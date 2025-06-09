{
  "name": "line-job-bot",
  "version": "1.0.0",
  "description": "LINE Bot ส่ง Flex Message เมื่อพิมพ์ รับงาน",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@line/bot-sdk": "^7.4.0",
    "express": "^4.18.2"
  }
}
# 54111
line-job-bot
const express = require('express');
const { Client, middleware } = require('@line/bot-sdk');

const config = {
  channelAccessToken: process.env.CHANNEL_ACCESS_TOKEN,
  channelSecret: process.env.CHANNEL_SECRET,
};

const app = express();
const client = new Client(config);
const userCooldowns = new Map();
const COOLDOWN_TIME = 5 * 60 * 1000; // 5 นาที

app.post('/webhook', middleware(config), (req, res) => {
  Promise.all(req.body.events.map(handleEvent))
    .then(result => res.json(result))
    .catch(err => {
      console.error(err);
      res.status(500).end();
    });
});

async function handleEvent(event) {
  if (event.type !== 'message' || event.message.type !== 'text') {
    return null;
  }

  const userId = event.source.userId;
  const text = event.message.text.trim();

  if (text === 'รับงาน') {
    const now = Date.now();
    const lastSent = userCooldowns.get(userId) || 0;

    if (now - lastSent < COOLDOWN_TIME) {
      return client.replyMessage(event.replyToken, {
        type: 'text',
        text: 'กรุณารอสักครู่ก่อนส่งคำสั่งนี้อีกครั้งนะครับ'
      });
    }

    userCooldowns.set(userId, now);

    const flexMessage = {
      type: 'flex',
      altText: 'รายละเอียดงาน',
      contents: {
        type: 'bubble',
        body: {
          type: 'box',
          layout: 'vertical',
          contents: [
            {
              type: 'text',
              text: 'รับงาน',
              weight: 'bold',
              size: 'xl',
              margin: 'md'
            },
            {
              type: 'text',
              text: 'นี่คือรายละเอียดงานของคุณ',
              wrap: true,
              margin: 'md'
            },
            {
              type: 'button',
              action: {
                type: 'uri',
                label: 'ดูรายละเอียดเพิ่มเติม',
                uri: 'https://your-site.example.com/job-detail'
              },
              margin: 'md'
            }
          ]
        }
      }
    };

    return client.replyMessage(event.replyToken, flexMessage);
  }

  return client.replyMessage(event.replyToken, {
    type: 'text',
    text: 'พิมพ์ "รับงาน" เพื่อดูรายละเอียดงานครับ'
  });
}

const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
