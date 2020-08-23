# Handling Multiple Intents in Watson Assistant

By default, Watson Assistant handles one intent at a time. For example, if we have an intent #Transactions that handles transactions queries and a user asks "I want to check my Transactions",then the assistant jumps to the node that responds to this condition. What about multiple intents in the same query? It gets more complicated when a user asks multiple questions at the same time. Let's say a user asks "I want to check my Transactions and my current balance?", in this example we have 
two different intents, one reponsible for transactions and the other for checking the balance. We will see how we can tweak the assistant to give him the ability to handle multiple intents and thus make it smarter.

# How Intents works?
Intents in Watson assistant are triggered using a conficence score. Whenever a user asks a question, it is sent to every intent model that you have created, and each intent model returns a confidence score. The assistant statifsies the condition using the intent that has the highest confidence score.
Let's say we have 3 intent models : #Transactions, #Balance and #Greetings. If a user asks a questions regarding his balance then highest confidence score will be the #Balance intent, and thus the assistant triggers the node that uses the #Balance intent. If we go back to the previous example "I want to check my Transactions and my current balance", the assitant will detect the two intents but can only trigger one node using the the intent that has the highest confidence score. With the concept of confidence score, it's possible to make the assistant handle multi-intents at the same time.

# Implementing a Multi-Intent Dialog

Using the confidence score, Watson assitant can detect the intents, so in case the user has a query with multiple intents, we store these intents and then iterate through them. These are the 3 main steps:

1- Define a confidence score threshold value: We define a constant variable to capture the intents 

2- Capturing the intents that have a score above the threshold value: For example if our constant variable is 0.5, we focus on all captured intents that have a score above 0.5 and store them.

3- Iterate through the intents: Customize and adjust the dialog to make it loop through the captured intents one by one, and thus trigger their corresponding node to answer the user's multi-intent question.

## 1. Defining the confidence score threshold value
The confidence score help us to optimize which intents to focus whenever Watson Assistant detect them. If no intents were detected, the assistant shows "Irrelevant", which means that no intent has a confidence score above 0.2. We need to make sure to put fair confidence threshold value , not too low and not too high (If the value is too high then it will be very hard to detect the intents). A value that ranges between 0.4-0.6 should be enough.
As you can see here in the image below, we have created a context variable named "confidence threshold" in the first node (Greetings) and set its value to 0.4

IMAGE

## 2.Capturing and storing the intents
Capturing the intents means that we want to extract all the intents from the user's input that have a confidence score above the defined threshold value. We create an node that will be responsible of exracting and saving the intents using the "filter" function to get these intents using this expression <? intents.filter('intent', 'intent.confidence >= $confidence_threshold') ?>. In this image for example, we are checking if the context variable intents (where we are  storing our detected intents) is null. If it is, then we get and save all the intents that are above the defined confidence score value in the "$intents" variable. ($intents variable is like an array containing extracted intents)

IMAGE

## 3. Iterating Through the intents
We have our extracted intents stored in the $intents variable. Now we need to loop though this list that contains our intents so we can trigger their node correspondingly one by one. We create a 2 child nodes to our Extract intents node. The first one checks if the list ($intents) is empty, which means that the assistant has not extracted any intents. Here we can simply just reset $intents to null. The second node is triggered whenever the list $intents has intents and here we can start the iteration.
The iteration happens in the "Iterate Intents" node, and we have an "Out of intent" node that is triggered when there are no more intent in the $intents . In the Iterate Intents node we remove the first itmen from the list $intents using the expression<? $intents.remove(0) ?>, we should add a that checks if the intent list is not empty. After each iteration we will jump back to this node, and if we didn't put this condition and our intent list is empty, then the assistant will jump to the "Out of Intents" node. So this is similar to like how the "while loop" in programming works. As long as $intents is not empty keep going back to iterate Intent node







