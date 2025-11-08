# Stored Procedure 
Stored  procedure are at the container level and you it has to be written in JavaScript. 

```js
var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
}
```
## Bounded Execution
JavaScript could use a lot RU's so it limit the amount of time it's run  for. The default timeout is 5 seconds (bounded execution time) until it roll back the transaction for **ALL** JavaScript functions. 

To prevent you reaching this time limit you will need to breakdown large task and execute them one by one. The below example used count to keep track and one item get inserted at a time.

![](Images/Pasted%20image%2020251108163445.png)

```js
function bulkImport(items) {
    var container = getContext().getCollection();
    var containerLink = container.getSelfLink();
    
    var count = 0;
    
    if (!items) throw new Error("The array is undefined or null.");
    
    var itemsLength = items.length;
    if (itemsLength == 0) {
        getContext().getResponse().setBody(0);
    }
    
    // call callback function once insertion is done
    tryCreate(items[count], callback);
    
    function tryCreate(item, callback) {
        var isAccepted = container.createDocument(containerLink, item, callback);
        
        //createDocument will return false if excution time exceed timeout
        //if it's not accepted it will retrun the count and function can 
        //resume where it left of in a new function iteration with 
        //with a rest timeout of 5 seconds
        
        if (!isAccepted) getContext().getResponse().setBody(count);
    }
    
    function callback(err, item, options) {
        if (err) throw err;  
        
        //Increment the count so the next item can be added   
        count++;
        
        if (count >= itemsLength) {
            getContext().getResponse().setBody(count);
        } else {
            tryCreate(items[count], callback);
        }
    }
}
```
## Triggers

> [!NOTE] 
> Triggers aren't automatically executed. They must be specified for each database operation where you want them to execute.

**Pre-triggers** are executed before modifying a database item. Pre triggers can't have any input, but a good example for pre triggers is to validate input before it get saved to a database. 

```js
function validateToDoItemTimestamp() {
    var context = getContext();
    var request = context.getRequest();

    // item to be created in the current operation
    var itemToCreate = request.getBody();

    // validate properties
    if (!("timestamp" in itemToCreate)) {
        var ts = new Date();
        itemToCreate["timestamp"] = ts.getTime();
    }

    // update the item that will be created
    request.setBody(itemToCreate);
}
```

Here is you can specify when you want a pre trigger to run, but honestly looks useless to me might as well do this C#.

```c#
dynamic newItem = new
{
    category = "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
};

await client.GetContainer("database", "container").CreateItemAsync(newItem, null, new ItemRequestOptions { PreTriggers = new List<string> { "trgPreValidateToDoItemTimestamp" } });
```

**Post-triggers** are executed after modifying a database item.  

This trigger queries for the metadata item and updates it with details about the newly created item.
```js
function updateMetadata() {
    var context = getContext();
    var container = context.getCollection();
    var response = context.getResponse();
    
    // item that was created
    var createdItem = response.getBody();
    
    // query for metadata document
    var filterQuery = 'SELECT * FROM root r WHERE r.id = "_metadata"';
    var accept = container.queryDocuments(container.getSelfLink(), filterQuery,
        updateMetadataCallback);
    if(!accept) throw "Unable to update metadata, abort";
    
    function updateMetadataCallback(err, items, responseOptions) {
        if(err) throw new Error("Error" + err.message);
        
        if(items.length != 1) throw 'Unable to find metadata document';
        
        var metadataItem = items[0];
        
        // update metadata
        metadataItem.createdItems += 1;
        metadataItem.createdNames += " " + createdItem.id;
        var accept = container.replaceDocument(metadataItem._self,
            metadataItem, function(err, itemReplaced) {
                    if(err) throw "Unable to update metadata, abort";
            });
            
        if(!accept) throw "Unable to update metadata, abort";
        return;
    }
}
```

Again, you have to also specify when you want the post trigger to execute. 

```c#
var newItem = { 
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};

await client.GetContainer("database", "container").CreateItemAsync(newItem, null, new ItemRequestOptions { PostTriggers = new List<string> { "trgPostUpdateMetadata" } });
```

> [!NOTE] 
> The post-trigger runs as part of the same transaction for the underlying item itself. An exception during the post-trigger execution fails the whole transaction. Anything committed is rolled back and an exception is returned.
## User Defined Functions 
Extends azure cosmos db SQP API query language grammar with your own business logic

```js
/**
 * Calculates the VAT for a given amount in GBP.
 * @param {number} amount - The base amount (before tax)
 * @param {number} [rate=0.2] - The VAT rate (default 20%)
 * @returns {number} - The amount including VAT
 */
function calculateVAT(amount, rate) {
    if (amount == null || amount < 0) {
        throw new Error("Amount must be a positive number");
    }
    
    // Default VAT rate in the UK is 20%
    if (rate == null) {
        rate = 0.2;
    }
    
    return amount + (amount * rate);
}

```

```sql
SELECT c.productName,
       c.price,
       udf.calculateVAT(c.price) AS priceWithVAT
FROM c
```
