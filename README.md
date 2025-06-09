const express = require('express');
const line = require('@line/bot-sdk');

const app = express();

// กำหนดค่าจากตัวแปรสภาพแวดล้อม (Environment Variables)
const config = {
  channelAccessToken: process.env.LINE_ACCESS_TOKEN,
  channelSecret: process.env.LINE_CHANNEL_SECRET,
};

const client = new line.Client(config);

// สร้าง webhook endpoint
app.post('/webhook', line.middleware(config), (req, res) => {
  Promise.all(req.body.events.map(handleEvent))
    .then(result => res.json(result));
});

// ฟังก์ชันสำหรับตอบกลับข้อความ
function handleEvent(event) {
  if (event.type !== 'message' || event.message.type !== 'text') {
    return Promise.resolve(null);
  }

  // ตอบกลับข้อความที่ผู้ใช้พิมพ์
  return client.replyMessage(event.replyToken, {
    type: 'text',
    text: `คุณพิมพ์ว่า: ${event.message.text}`,
  });
}

// เริ่มเซิร์ฟเวอร์
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`กำลังฟังที่พอร์ต ${PORT}`);
});
