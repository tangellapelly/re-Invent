## Building a gas station chatbot with AWS Lex, Lambda and HERE services:

In this tutorial, we will deploy a small chatbot with AWS Lex and HERE Location Services to help users find gas stations near a specified location. We will explore the combination of AWS services (Lex, Lambda & Payments), HERE Location services API's and how they can be stitched together to create a powerful, scalable and secure application. To complete this tutorial, you will need: An AWS Account, basic knowledge of the AWS UI, account with developer.here.com (created through the HERE AWS Marketplace Listing) ,and a basic understanding of NodeJS and Javascript.

# Step 1: Creating a HERE Account through the AWS Marketplace Listing

/// please add screenshots and walk the developer through the sign up flow

# Step 1: Deploy Lex bot <-- please renumber these

Login to your AWS console, search for Lex and click on it. Now, click on the create button to get started. On the next screen, click "Custom Bot", fill in the details as shown in the screenshot below.

![alt text](screenshots/step1.png "Step 1: Deploy Lex bot")

Once you fill in the all the required details click on *Create*
button to navigate to the next screen. 

![alt text](screenshots/step2.png "Create Intent")

On the next screen you will create an "Intent". An Intent is a objective or action the user wants to achieve. Think of an intent as the specific reason a user would want to send a message to your bot. Lets create an intent and call it "gasStationNearBy". 

![alt text](screenshots/step3.png "Create Intent")

Once you’ve done that, you will be presented with another screen with a whole bunch of input options. Lets focus on the "Slots" section first.

![alt text](screenshots/step4.png "Slots")

You can think of "Slots" as variables in Lex. If someone was ordering a pizza, the Slots to include would be things like {SIZE}, {PIZZA_TYPE} and {DELIVERY_ADDRESS}. For our chatbot, we will have one slot called "{gpsCodes}". There are essentailly two types, "custom" and "built-in". For our chatbot we will be creating the "custom" slot. I encourage you to read more information about slots in the <a href="https://docs.aws.amazon.com/lex/latest/dg/howitworks-builtins-slots.html">Amazon Lex</a> documentation.

To create a custom slot, go ahead added new slot type from the lefthand side menu, you will find a dialog box where you  will create slot type(As shown in the screenshot below)

![alt text](screenshots/step5.png "Creating Custom Slots")

Once you click on the *create slot type* you will be presented with a bunch of options. Follow the screenshot below to complete the setup and click on the *Add slot to intent* button.

![alt text](screenshots/step6.png "Creating Custom Slots")

Once the slot successfully saved, you can find the "*gpsCodes*" slot under the custom slot as shown in the screenshot below.

![alt text](screenshots/step7.png "selecting the custom slot to the intent")

Now that we have added our custom slot, let us go ahead and setup the slot for our intent. As you can see in the following screenshot, we give a name to our custom slot. From the slot type dropdown, select "gpsCodes" under the "*Custom slot types*" and then give a prompt like "_which gps coordinates?_" 

Click on the "Add" button under the settings to apply the slot to the current intent. Finally, click on the "*Save Intent*" button located on bottom of the page.

![alt text](screenshots/step8.png "applying the slot to the intent")

Now to make the Lex bot more flexible we add in variations of utterances. Lets add the following utterances.

* "gas stations near {gpsCodes}"
* "petrol stations near {gpsCodes}"
* "fuel stations near {gpsCodes}"

Here is what it should look like.
![alt text](screenshots/step9.png "Adding uttetances")

Note: For the Fulfillment section, you have the option to either send the intent to a lambda function or return parameters to client. For now, lets click Return Parameters to Client and click save. We’re going to check on our bot before moving over to Amazon Lambda and creating the business logic of our chatbot. Scroll down and save your intent.

![alt text](screenshots/step10.png "Adding Fulfillment")

And now for a bit of fun! On the top right of the dashboard, click Build. This will initialize your chatbot and allow you to begin testing. A chat widget should pop up on the right side of the screen. Try it out!

Type: "gas stations near 12.972442,77.580643"


Press return and your response should look like this following image.

![alt text](screenshots/step11.png "Building and testing the bot")
Awesome, our Lex chatbot is almost ready to go. Our next step is to attach a Lambda function to it.

Our Lambda will simply receive the slots and their values and return them in a way that Lex understands.

Normally at this point we would point you to the HERE Serverless Application Repository, but our Places Lambda function is not compatible with Lex, but it is compatible with many many other use cases.  Check it out when you get a moment [Please insert link here.]

## Step 2: Deploy a Lambda Function

In your AWS console, go to Amazon Lambda and select "Build a Function from Scratch". In the next screen, you will have the option of building your custom function or selecting a blueprint. The blueprints are great examples and I highly recommend checking them out to enhance your knowledge. 

For our purposes, "Select Author From Scratch" and fill out the form with your function name [Please give an example of what they should call the function], runtime environment (Node.js 10.x) and select the "*create function*" button to create the lambda function.


![alt text](screenshots/step12.png "creating lambda function")

On this next Screen you will be treated to your lambda function, the designer tool, and some dashboard items for testing and monitoring your function. The following is the lambda function that we want to create. Copy and paste this code into ..... (please complete...)

```javascript
'use strict';
const axios = require('axios')     
function close(sessionAttributes, fulfillmentState, message) {
    return {
        sessionAttributes,
        dialogAction: {
            type: 'Close',
            fulfillmentState,
            message,
        },
    };
}
 
// --------------- Events -----------------------
 
async function dispatch(intentRequest, callback) {
    console.log(`request received for userId=${intentRequest.userId}, intentName=${intentRequest.currentIntent.name}`);
    const sessionAttributes = intentRequest.sessionAttributes;
    const slots = intentRequest.currentIntent.slots;
    const city = slots.city;
 
    if(slots.gpsCodes){
        
          const gpsCodes = slots.gpsCodes;
    const codes = gpsCodes.split(',');
       // API to get the list of gas stations
              let gasStation = axios.get(`https://places.demo.api.here.com/places/v1/discover/search?at=${codes[0].trim()},${codes[1].trim()}&q=petrol-station&app_id=MMRyT9PioGx6DeImyPie&app_code=SB7YD1dqPH40vz-lSJE19g`, {}).then((data)=>{
                 
                let gasStationNames= data.data.results.items.map((it)=>{
                    
                    return ` *${it.title}* is at _${it.vicinity.replace("<br/>"," ")}_`+"\n"
                })
                const reducer = (accumulator, currentValue) => accumulator + currentValue;

                callback(close(sessionAttributes, 'Fulfilled',
                {'contentType': 'PlainText', 'content': `Okay, HERE are the list of gas stations ${"\n"}${gasStationNames.reduce(reducer)}`}));
                
            }).catch((e)=>{
        console.log(e)
    })  
    } else {
         // API to convert the address to lat and lng 
    const res = await axios.get(`https://geocoder.api.here.com/6.2/geocode.json?app_id=MMRyT9PioGx6DeImyPie&app_code=SB7YD1dqPH40vz-lSJE19g&searchtext=${city}`, {}).then((res)=>{
            let lat = res.data.Response.View[0].Result[0].Location.DisplayPosition.Latitude;
            let lng = res.data.Response.View[0].Result[0].Location.DisplayPosition.Longitude
            
            // API to get the list of gas stations
              let gasStation = axios.get(`https://places.demo.api.here.com/places/v1/discover/search?at=${lat},${lng}&q=petrol-station&app_id=MMRyT9PioGx6DeImyPie&app_code=SB7YD1dqPH40vz-lSJE19g`, {}).then((data)=>{
                 
                let gasStationNames= data.data.results.items.map((it)=>{
                    
                    return ` *${it.title}* is at _${it.vicinity.replace("<br/>"," ")}_`+"\n"
                })
                const reducer = (accumulator, currentValue) => accumulator + currentValue;

                callback(close(sessionAttributes, 'Fulfilled',
                {'contentType': 'PlainText', 'content': `Okay, HERE are the list of gas stations near by ${city}${"\n"}${gasStationNames.reduce(reducer)}`}));
                
            })
           
    }).catch((e)=>{
        console.log(e)
    })
   
   
    }
    
}




// --------------- Main handler -----------------------
 
// Route the incoming request based on intent.
// The JSON body of the request is provided in the event slot.
exports.handler = (event, context, callback) => {
    try {
        dispatch(event,
            (response) => {
                callback(null, response);
            });
    } catch (err) {
        callback(err);
    }
};
```


The first time you hit Test, you will be asked to supply a JSON snippet describing what the test input should be.

``` json
{
  "messageVersion": "1.0",
  "invocationSource": "FulfillmentCodeHook",
  "userId": "user-1",
  "sessionAttributes": {},
  "bot": {
    "name": "HerePlacesBot",
    "alias": "$LATEST",
    "version": "$LATEST"
  },
  "outputDialogMode": "Text",
  "currentIntent": {
    "name": "HerePlacesBot",
    "slots": {
      "gpsCodes": "18.000055,79.588165"
    },
    "confirmationStatus": "None"
  }
}
```

