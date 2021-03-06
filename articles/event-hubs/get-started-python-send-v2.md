---
title: Send or receive events from Azure Event Hubs using Python (latest)
description: This article provides a walkthrough for creating a Python application that sends/receives events to/from Azure Event Hubs using the latest azure-eventhub version 5 package.
services: event-hubs
author: spelluru

ms.service: event-hubs
ms.workload: core
ms.topic: quickstart
ms.date: 01/30/2020
ms.author: spelluru

---

# Send events to or receive events from event hubs by using Python (azure-eventhub version 5)

Azure Event Hubs is a big data streaming platform and event-ingestion service that can receive and process millions of events per second. Event hubs can process and store events, data, or telemetry that's produced by distributed software and devices. Data that's sent to an event hub can be transformed and stored by using any real-time analytics provider or batching/storage adapters. For more information, see [Event Hubs overview](event-hubs-about.md) and [Event Hubs features](event-hubs-features.md).

This quickstart describes how to create Python applications that can send events to or receive events from an event hub.

> [!IMPORTANT]
> This quickstart uses version 5 of the Azure Event Hubs Python SDK. For a quick start that uses version 1 of the Python SDK, see [this article](event-hubs-python-get-started-send.md). 

## Prerequisites

To complete this quickstart, you need the following prerequisites:

- An Azure subscription. If you don't have one, [create a free account](https://azure.microsoft.com/free/) before you begin.
- An active Event Hubs namespace and event hub. To create them, follow the instructions at [Quickstart: Create an event hub by using the Azure portal](event-hubs-create.md). Record the namespace and event hub names to use later in this quickstart.
- The shared access key name and primary key value for your Event Hubs namespace. Get the access key name and value by following the instructions at [Get an event hubs connection string](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). The default access key name is *RootManageSharedAccessKey*. Record the key name and the primary key value to use later in this quickstart.
- Python 2.7 or 3.5 or later, with PIP installed and updated.
- The Python package for Event Hubs. 

    To install the package, run this command in a command prompt that has Python in its path:

    ```cmd
    pip install azure-eventhub
    ```

    Install the following package for receiving the events by using Azure Blob storage as the checkpoint store:

    ```cmd
    pip install azure-eventhub-checkpointstoreblob-aio
    ```

## Send events
In this section, you create a Python script to send events to the event hub that you created earlier.

1. Open your favorite Python editor, such as [Visual Studio Code](https://code.visualstudio.com/).
2. Create a script called *send.py*. This script sends a batch of events to the event hub that you created earlier.
3. Paste the following code into *send.py*:

    ```python
    import asyncio
    from azure.eventhub.aio import EventHubProducerClient
    from azure.eventhub import EventData

    async def run():
        # Create a producer client to send messages to the event hub.
        # Specify a connection string to your event hubs namespace and
     	    # the event hub name.
        producer = EventHubProducerClient.from_connection_string(conn_str="EVENT HUBS NAMESPACE - CONNECTION STRING", eventhub_name="EVENT HUB NAME")
        async with producer:
            # Create a batch.
            event_data_batch = await producer.create_batch()

            # Add events to the batch.
            event_data_batch.add(EventData('First event '))
            event_data_batch.add(EventData('Second event'))
            event_data_batch.add(EventData('Third event'))

            # Send the batch of events to the event hub.
            await producer.send_batch(event_data_batch)

    loop = asyncio.get_event_loop()
    loop.run_until_complete(run())

    ```

    > [!NOTE]
    > For the complete source code, including informational comments, go to the [GitHub send_async.py page](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub/samples/async_samples/send_async.py).

## Receive events
This quickstart uses Azure Blob storage as a checkpoint store. The checkpoint store is used to persist checkpoints (that is, the last read positions).  

### Create an Azure storage account and a blob container
Create an Azure storage account and a blob container in it by doing the following steps:

1. [Create an Azure Storage account](../storage/common/storage-account-create.md?tabs=azure-portal)
2. [Create a blob container](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)
3. [Get the connection string to the storage account](../storage/common/storage-configure-connection-string.md?#view-and-copy-a-connection-string)

Be sure to record the connection string and container name for later use in the receive code.


### Create a Python script to receive events

In this section, you create a Python script to receive events from your event hub:

1. Open your favorite Python editor, such as [Visual Studio Code](https://code.visualstudio.com/).
2. Create a script called *recv.py*.
3. Paste the following code into *recv.py*:

    ```python
    import asyncio
    from azure.eventhub.aio import EventHubConsumerClient
    from azure.eventhub.extensions.checkpointstoreblobaio import BlobCheckpointStore


    async def on_event(partition_context, event):
        # Print the event data.
        print("Received the event: \"{}\" from the partition with ID: \"{}\"".format(event.body_as_str(encoding='UTF-8'), partition_context.partition_id))

        # Update the checkpoint so that the program doesn't read the events
        # that it has already read when you run it next time.
        await partition_context.update_checkpoint(event)

    async def main():
        # Create an Azure blob checkpoint store to store the checkpoints.
        checkpoint_store = BlobCheckpointStore.from_connection_string("AZURE STORAGE CONNECTION STRING", "BLOB CONTAINER NAME")

        # Create a consumer client for the event hub.
        client = EventHubConsumerClient.from_connection_string("EVENT HUBS NAMESPACE CONNECTION STRING", consumer_group="$Default", eventhub_name="EVENT HUB NAME", checkpoint_store=checkpoint_store)
        async with client:
            # Call the receive method.
            await client.receive(on_event=on_event)

    if __name__ == '__main__':
        loop = asyncio.get_event_loop()
        # Run the main method.
        loop.run_until_complete(main())    
    ```

    > [!NOTE]
    > For the complete source code, including additional informational comments, go to the [GitHub recv_with_checkpoint_store_async.py 
page](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub/samples/async_samples/recv_with_checkpoint_store_async.py).


### Run the receiver app

To run the script, open a command prompt that has Python in its path, and then run this command:

```bash
python recv.py
```

### Run the sender app

To run the script, open a command prompt that has Python in its path, and then run this command:

```bash
python send.py
```

The receiver window should display the messages that were sent to the event hub.


## Next steps
In this quickstart, you've sent and received events asynchronously. To learn how to send and receive events synchronously, go to the [GitHub sync_samples page](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub/samples/sync_samples).

For all the samples (both synchronous and asynchronous) on GitHub, go to [Azure Event Hubs client library for Python samples](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub/samples).
