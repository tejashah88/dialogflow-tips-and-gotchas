# Disclaimer
The list of tips and gotchas are by no means exhaustive, nor are they meant to shine a negative light upon Dialogflow. Dialogflow is a good platform (in my opinion) for easily building multi-platforms chat/voice bots and recommended for new projects. It's really meant for myself and other fellow developers who've worked with the platform and found some inconsistencies not documented anywhere else. If you find a tip or quirk to add, feel free to contribute with a pull request.

# Tips/Gotchas

## Platform of action source is null or undefined?
When you send queries from the Dialogflow test console, the source parameter from the request may not be specified.

## Dealing with @sys.any taking priority over other queries (Can't cancel with @sys.any?)
One of the main gotchas of using @sys.any is that it takes priority over any other command or entity, including when the user wants to cancel an ongoing action. Your best bet is to process the raw query and respond from there. I'd recommend to either implement a fuzzy search or neural network classifier to separate the "cancel" phrases from the rest of the implemented phrases. I implemented a working proof of concept for one of my earlier projects [here](https://github.com/tejashah88/eznet/tree/master/src/nlp), but the general idea is to take an export of your dialogflow assets, collects the implemented phrases and cleans it up for processing. Then, either for fuzzy searching for neural network classifying, load the positive and negative examples, and make sure to process the raw query before having a webhook-handled intent process it.

## 5 second timeout for webhook-based intents
Dialogflow enforces a 5 second timeout period for the backend to respond before taking over and using the default responses specified in the web console. This can be problematic when handling potentially long queries that need access to external resources, as the user can easily become confused and think that the bot is not working. The best solution is to use a caching service for this problem (a database can work but you risk wasting time the database to retrieve the results from storage as opposed to an in-memory solution). Redis is a good option for this scenario, but you may choose whatever caching service works best for you.

## Proper context handling
I'm not entirely sure if there's a better way of doing this, but for handing cases where an intent can have some required parameters ommited if they were previously saved in a context object, it's best to create three different intents to handle the following scenarios:
1. intent was invoked with explicitly given parameter
2. intent was invoked without explicitly given parameter but a context object satisfies said parameter
3. intent was invoked without explicitly given parameter and no context object satisfies said parameter

Scenario 1 covers the standard scenario for basic bot building. The parameter should be marked as "required". Scenario 2 handles an ommited parameter that's given in a context object. This is useful for when you don't want to make the user refer to a specific name every time he/she talks to the bot (i.e. "what's the price of *that* item"). You would also make the expected parameter as "optional" since the context will potentially have a valid value to fill it in. Scenario 3 handles when an ommited parameter isn't found in the context object. In this case, the parameter should be marked as "required".

## Fitting parameters into a dynamic list of unknown values? Enter fuzzy searching!
If you have a parameter that can only fit a list of defined values, but that list of values can change over time, you can use a combination of the @sys.any type and fuzzy searching for this issue. The @sys.any allows free form input but the fuzzy searching allows the webhook to narrow down the input into a set of expected responses or reject the input if a suitable match can't be found. It's also perfect for handling spelling errors with the input. This is different from having an entity since the values of entities do not change over time.

## Dealing with button actions in message templates for Facebook Messenger
When using the button actions from the [message templates](https://developers.facebook.com/docs/messenger-platform/send-messages/template/generic), there's a barely noticeable quirk that separates the apparent text and the *actual* text being sent to the bot. The following is a button JSON object used for interacting the message template with Dialogflow.

```json
"buttons": [
  {
    "type": "postback",
    "title": "Start chatting",
    "payload": "Actually start chatting"
  }              
]
```

Most of us would think that the title, which is the text visibly sent to the bot, would be sent to Dialogflow, but it's actually what's in the payload that's being sent. In this case, you'll need to have an intent that handles whatever is in the payload. I'd recommend using template annotations, disabling ML and using some odd symbols like "<>" or "{}" to signify that the request is coming from a message template.
