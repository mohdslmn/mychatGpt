
npm init -y 

this will generate a package.json file.

for server side applications downoalding couple of dependencies:

1. cors : use for cross-origin request
2. dotenv : use for secure enviroment variables
3. Express : backened frame work
4. nodemon: when we want keep our application running while we make changes
5. openai : for generating AI app

login with openAI click on profile => view API keys => genrate new key

copy that API in .env file which is created in root folder.



import express, { json } from "express";
import * as dotenv from "dotenv";
import cors from "cors";
import {Configuration , openAIApi} from 'openai';

dotenv.config();

const configuration = new Configuration({
    apiKey : process.env.OPENAI_API_KEY ,
}) ;

const openai = new openAIApi(configuration);
const app = express();

app.use(cors());
//allowed our server to be called from the frontend
app.use(express.json())
//allow us to pass json from the frontend to the backened

//from get we get data from frontend to backend
app.get("/" , async(req , res) => {
    res.status(200).send({
        message: "Hello from CodeX",
    })
});

//from post we get payload or body
app.post('/',async (req, res) => {
    try{
     const prompt = req.body.prompt;
     const response = await openai.createCompletion({
        model: "text-davinci-003",
      prompt: `${prompt}`,
      temperature: 0, // Higher values means the model will take more risks.
      max_tokens: 3000, // The maximum number of tokens to generate in the completion. Most models have a context length of 2048 tokens (except for the newest models, which support 4096).
      top_p: 1, // alternative to sampling with temperature, called nucleus sampling
      frequency_penalty: 0.5, // Number between -2.0 and 2.0. Positive values penalize new tokens based on their existing frequency in the text so far, decreasing the model's likelihood to repeat the same line verbatim.
      presence_penalty: 0, // Number between -2.0 and 2.0. Positive values penalize new tokens based on whether they appear in the text so far, increasing the model's likelihood to talk about new topics.
     });
       //send this data to frontend
    res.status(200).send({
        bot: response.data.choices[0].text
    })
    }
  
    catch (error){
        console.log(error);
        res.status(500).send({error})
    }
}) 
//we should make sure that  our server always listens for a new request

app.listen(5000, () => console.log('Server is running on http://localhost:5000'));


//scrip.js

const handleSubmit = async (e) => {
  e.preventDefault();

  //get data type into form element
  const data = new FormData(form);

  //user's chatsripe
  chat_container.innerHTML +=  chatStripe(false , data.get('prompt'));

  //to clear textarea input
  form.reset();

  //bot's chatstripe
  const uniqueId =generateUniqueId();
  // true because AI is typing
  chat_container.innerHTML +=  chatStripe(true , " " , uniqueId);

  //keep scrolling
  chat_container.scrollTop = chat_container.scrollHeight;

  const messageDiv = document.getElementById(uniqueId);
  loader(messageDiv); 

  //fetch data from server -> bot's response
  const response = await fetch("http://localhost:5000/", {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json' ,
    },
    body: JSON.stringify({
      prompt: data.get('prompt')
    })
  })
  clearInterval(loadInterval);
  //we dont know at which point of loading are we at one dot , 2 dot , or 3 , we have to make empty to print a message
  messageDiv.innerHTML = "";

  if(response.ok){
    //this is giving actual response from backend
    const data = await response.json() ;
    console.log(data);
    //to parse the data
    const parsedData = data.bot.trim();

    typeText(messageDiv , parsedData) ;
  
  }
  else{
    const error = await response.text();
    messageDiv.innerHTML = "Something Went Wrong";
    alert(error);
  }

}

// serve.js

import express from 'express'
import * as dotenv from 'dotenv'
import cors from 'cors';
import openai from 'openai';


dotenv.config();
// console.log(process.env.OPENAI_API_KEY);

const openaiInstance = new openai({ apiKey: process.env.OPENAI_API_KEY });

const app = express()
app.use(cors())
app.use(express.json())

app.get('/', async (req, res) => {
  res.status(200).send({
    message: 'Hello from CodeX!'
  })
})

app.post('/', async (req, res) => {
  try {
    const prompt = req.body.prompt;
    console.log("prompt body:", prompt);


    const response = await openaiInstance.createCompletion({
      model: "text-davinci-003",
      prompt: `${prompt}`,
      temperature: 0, // Higher values means the model will take more risks.
      max_tokens: 3000, // The maximum number of tokens to generate in the completion. Most models have a context length of 2048 tokens (except for the newest models, which support 4096).
      top_p: 1, // alternative to sampling with temperature, called nucleus sampling
      frequency_penalty: 0.5, // Number between -2.0 and 2.0. Positive values penalize new tokens based on their existing frequency in the text so far, decreasing the model's likelihood to repeat the same line verbatim.
      presence_penalty: 0, // Number between -2.0 and 2.0. Positive values penalize new tokens based on whether they appear in the text so far, increasing the model's likelihood to talk about new topics.
    });
    

    res.status(200).send({
      bot: response.data.choices[0].text
    });

  } catch (error) {
    console.error('Error:', error);
    res.status(500).send({ error: 'Something went wrong', message: error.message, stack: error.stack });
    
  }
})

app.listen(5000, () => console.log('AI server started on http://localhost:5000'))

//script.js

import bot from "./assets/bot.svg";
//importing icon from assets
import user from "./assets/user.svg";

const form = document.querySelector("form"); //direct selecting the tag bcoz there is only one
const chat_container = document.querySelector("#chat_container");

let loadInterval;

function loader(element) {
  element.textContent = ""; // to ensure that it is empty at start

  loadInterval = setInterval(() => {
    element.textContent += ".";

    if (element.textContent === "....") {
      element.textContent = "";
    }
  }, 300);
}

//while producing result text appears letter by letter
//designing this

function typeText(element, text) {
  let index = 0;
  let inteval = setInterval(() => {
    //it means we are still typing
    if (index < text.length) {
      element.innerHTML += text.charAt(index); //will give character under specific index
      index++;
    } else {
      clearInterval(inteval);
    }
  }, 20);
}
//generate unique ID for every message

function generateUniqueId() {
  const timeStamp = Date.now();
  const randomNumber = Math.random();
  const hexadecimalString = randomNumber.toString(16);

  return `id-{$timeStamp}-{$hexadecimalString}`;
}

//each message has stripe such as : text generated by AI is in light gray color , icon of prompt and icon of text genrated by AI is different

function chatStripe(isAi, value, uniqueId) {
  return (
    `
      <div class = "wrapper ${isAi && "ai"}">
       <div class = "chat">
        <div class = "profile">
         <img 
           src = "${isAi ? bot : user}"
           alt = "${isAi ? "bot" : "user"}"
         />
        </div>
         <div class = "message" id = ${uniqueId}>${value} </div>
        </div>
      </div>
    `
    );
}

// to trigger AI generated response
const handleSubmit = async (e) => {
  e.preventDefault();

  //get data type into form element
  const data = new FormData(form);

  //user's chatsripe
  chat_container.innerHTML +=  chatStripe(false , data.get('prompt'));

  //to clear textarea input
  form.reset();

  //bot's chatstripe
  const uniqueId =generateUniqueId();
  // true because AI is typing
  chat_container.innerHTML +=  chatStripe(true , " " , uniqueId);

  //keep scrolling
  chat_container.scrollTop = chat_container.scrollHeight;

  const messageDiv = document.getElementById(uniqueId);
  loader(messageDiv); 

  //fetch data from server -> bot's response
  const response = await fetch("http://localhost:5000/", {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json' ,
    },
    body: JSON.stringify({
      prompt: data.get('prompt')
    })
  })
  console.log(response);
  clearInterval(loadInterval)
  //we dont know at which point of loading are we at one dot , 2 dot , or 3 , we have to make empty to print a message
  messageDiv.innerHTML = " ";

  if(response.ok){
    //this is giving actual response from backend
    const data = await response.json() ;
     console.log(data);
    //to parse the data
    const parsedData = data.bot.trim();
    console.log(parsedData);


    typeText(messageDiv , parsedData) ;
  
  }
  else{
    const error = await response.text();
    messageDiv.innerHTML = "Something Went Wrong";
    // alert(error);
    console.log(error);
  }

}

form.addEventListener("submit", handleSubmit);
//when we press enter key
form.addEventListener("keyup" ,(e) => {
  if(e.keyCode === 13){
    handleSubmit(e);
  }
});



