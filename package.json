const express = require('express');
const bodyParser = require('body-parser');
const twilio = require('twilio');
const OpenAI = require('openai');
require('dotenv').config();

const app = express();
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

const accountSid = process.env.TWILIO_ACCOUNT_SID;
const authToken = process.env.TWILIO_AUTH_TOKEN;
const twilioClient = twilio(accountSid, authToken);
const fromNumber = process.env.TWILIO_WHATSAPP_FROM;

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const systemPrompt = `You are a professional UK plumber's receptionist. Your name is Theo. Be friendly, capture customer details (name, phone, address, the issue), and help them book an appointment. If urgent, say someone will call urgently. Keep responses short.`;

async function getAIResponse(userMessage) {
  try {
    const completion = await openai.chat.completions.create({
      model: "gpt-4",
      messages: [
        { role: "system", content: systemPrompt },
        { role: "user", content: userMessage }
      ],
      max_tokens: 150
    });
    return completion.choices[0].message.content;
  } catch (error) {
    console.error("OpenAI Error:", error);
    return "Thanks! We'll call you shortly.";
  }
}

app.post('/webhook', async (req, res) => {
  const incomingMsg = req.body.Body;
  const from = req.body.From;
  console.log("From " + from + ": " + incomingMsg);
  
  const response = await getAIResponse(incomingMsg);
  
  try {
    await twilioClient.messages.create({
      body: response,
      from: fromNumber,
      to: from
    });
  } catch (error) {
    console.error("Twilio Error:", error);
  }
  res.sendStatus(200);
});

app.get("/", (req, res) => { res.send("Theo AI Bot running!"); });
app.listen(3000, function() { console.log("Server on port 3000"); });
