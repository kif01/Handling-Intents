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





