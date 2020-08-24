# Handling Multiple Intents in Watson Assistant
By default, Watson Assistant handles one intent at a time. For example, if we have an intent #Transactions that handles transaction queries and a user asks "I want to check my Transactions", then the assistant jumps to the node that responds to this condition. What about multiple intents in the same query? It gets more complicated when a user asks multiple questions at the same time. For example, if a user asks "I want to check my Transactions and my current balance?", we have two different intents. One responsible for transactions and the other for checking the balance. We will see how we can tweak the assistant to give him the ability to handle multiple intents and thus make it smarter.

# How Intents works?
Intents in Watson assistant are triggered using a confidence score. Whenever a user asks a question, it's sent to every intent model that you have created, and each intent model returns a confidence score. The assistant satisfies the condition using the intent that has the highest confidence score.</br>
Let's say we have 3 intent models: #Transactions, #Balance and #Greetings. If a user asks a question regarding his balance, then the highest confidence score will be the #Balance intent, and thus the assistant triggers the node that uses the #Balance intent. If we go back to the previous example "I want to check my Transactions and my current balance", the assistant will detect the two intents but can only trigger one node using the intent that has the highest confidence score. With the concept of confidence score, it's possible to make the assistant handle multi-intents at the same time.

# Implementing a Multi-Intent Dialog

Using the confidence score, Watson assitant can detect the intents, so in case the user has a query with multiple intents, we store these intents and then iterate through them. These are the 3 main steps: </br>

1- Define a confidence score threshold value: We define a constant variable to capture the intents. </br>

2- Capturing the intents that have a score above the threshold value: For example, if our constant variable is 0.5, we focus on all captured intents that have a score above 0.5 and store them. </br>

3- Iterate through the intents: Customize and adjust the dialog to make it loop through the captured intents one by one, and thus trigger their corresponding node to answer the user's multi-intent question. </br>

## 1. Defining the confidence score threshold value
The confidence score helps us to optimize which intents to focus whenever Watson Assistant detects them. If no intents were detected, the assistant shows "Irrelevant", which means that no intent has a confidence score above 0.2. We need to make sure to put fair confidence threshold value, not too low and not too high (If the value is too high then it will be very hard to detect the intents). A value that ranges between 0.4-0.6 should be enough.As you can see here in the image below, we have created a context variable named "confidence threshold" in the first node (Greetings) and set its value to 0.4.

<img width="791" alt="Screen Shot 2020-08-20 at 3 02 01 PM" src="https://user-images.githubusercontent.com/15332386/91048378-a6d2be00-e62c-11ea-9920-f3ebbea83e48.png">

## 2.Capturing and storing the intents
Capturing the intents means that we want to extract all the intents from the user's input that have a confidence score above the defined threshold value. We can create a node that is responsible for extracting and saving the intents using the "filter" function to get these intents with this expression <? intents.filter('intent', 'intent.confidence >= $confidence_threshold') ?>.  The image below shows how we are checking if the context variable $intents (where we are storing our detected intents) is null. If it is, then we get and save all the intents that are above the defined confidence score value in the "$intents" variable ($intents variable is like an array containing the extracted intents).

<img width="1232" alt="Screen Shot 2020-08-20 at 6 58 05 PM" src="https://user-images.githubusercontent.com/15332386/91048550-e4374b80-e62c-11ea-87fe-b81151618822.png">

We create 2 child nodes to our Extract intents node. The first one checks if the list $intents is empty. Here we can simply just reset $intents to null. The second node is triggered whenever the list $intents has intents so we can start the iteration.</br>
<img width="1245" alt="Screen Shot 2020-08-25 at 1 08 50 AM" src="https://user-images.githubusercontent.com/15332386/91096831-b9bab200-e66f-11ea-9e9d-6e929ed299c5.png">

## 3. Iterating Through the intents
We have our extracted intents stored in the $intents variable. Now we need to loop through this list that contains our intents so we can trigger their node correspondingly one by one.</br>
The iteration happens in the "Iterate Intents" node. In the Iterate Intents, we first add a condition that checks if the intent list is not empty since we will jump back to this node after each iteration, then we get the first item from the list $intents using the expression "<? $intents.get(0).intent ?>" and store it in a new context variable so we can use it later on (in the image below, the intent is stored in $current_intent). 
<img width="1232" alt="Screen Shot 2020-08-24 at 3 38 40 PM" src="https://user-images.githubusercontent.com/15332386/91071528-5b7ad880-e649-11ea-981b-72580669e901.png">
Once it's stored, we need to remove this item from the $intents list which takes place in the node called Decrement array. We remove the intent from the list using this expression "<? $intents.remove(0) ?>". Removing the intent is necessary so we don't get stuck in an infinite loop whenever we jump back to the Iterate Intents node. As you can see in the below image, once we remove the intent from the list we jump to Evaluate Intents which is just an entry point to the main dialogue (where we will be using the $current_intent to trigger its corresponding node). In other words, the Evaluate Intent node does nothing but is just a separation between the process of extracting the intents and the main dialogue.
<img width="1235" alt="Screen Shot 2020-08-24 at 9 58 33 PM" src="https://user-images.githubusercontent.com/15332386/91079358-ffb64c80-e654-11ea-841b-5d9f5d10b820.png">
If the list is empty, then the assistant will jump to the "Out of Intents" node (You can add this node at the bottom end of your dialog). This node just reset our list $intent to null. 
<img width="1204" alt="Screen Shot 2020-08-24 at 4 00 32 PM" src="https://user-images.githubusercontent.com/15332386/91072208-3470d680-e64a-11ea-8eec-d2089b5111b5.png">
There is one last step to modify which is using the $current_intent to trigger the dialogue nodes. Usually in Watson Assistant, if we want to use intents in the nodes we use #INTENT-NAME, but here we modified the way it works because we are working with a list that contains multiple intents. We are extracting the first item of the list $intents and store it in a context variable $current_intent. In this case, we need to use this context variable, so instead of putting #INTENT-NAME in the node, we put $CONTEXT-VARIABLE which is $current_intent. We add $current_intent on every node that we want to trigger. <br>
For example if $current_intent has the value "updateID ", instead of putting #updateID for the Update ID node, we put $current_intent=="updateID" so the node regarding the ID update can get triggered. In addition, every dialogue node that we want to trigger must have conditioned responses. In case the list $intents has more than one intent we show the response to the user and add a jump back to the Iterate Node to repeat the process and thus trigger the node for the other intent. If the list is empty (the intent that we currently have is the last intent in the list) then no need to add a jump so we just show the response. For example, if a user asks about his ID update and transactions, the assistant will first trigger the Update ID node and gives the response, but won't stop there. The list has another intent extracted which is transactions, thus the list $intents is not empty yet so the assistant will automatically repeat the process by jumping back to the node where we iterate through the list and thus eventually show the response for the transaction intent.
<img width="1228" alt="Screen Shot 2020-08-24 at 3 58 24 PM" src="https://user-images.githubusercontent.com/15332386/91072488-a8ab7a00-e64a-11ea-9db4-f32f07812207.png">

The modification that we created here works just like a while loop. We extract the intents from the user's query and store them in a context variable that is a list containing these items. We get the first item from the list, store it another context variable, and remove it from the list. Then we go and check which node uses this intent that it's stored in our context variable and trigger it. Once it's done we check, if the list still has items, then we go back to the iterating node, and the process gets repeated to trigger the other intents captured and stored in the list. 









