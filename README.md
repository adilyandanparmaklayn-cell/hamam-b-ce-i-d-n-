
import { TikTokLiveClient, EventType } from "piratetok-live-js";
import { createServer } from "http";
import { WebSocketServer } from "ws";

const PORT = process.env.PORT || 3000;
const server = createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hamam Bocegi Dugunu - TikTok baglantisi hazir");
});

const wss = new WebSocketServer({
  server,
  path: "/events"
});

function sendEvent(event) {
  const message = JSON.stringify(event);

  for (const socket of wss.clients) {
    if (socket.readyState === 1) {
      socket.send(message);
    }
  }
}

function getUsername(data) {
  return data.user?.uniqueId ||
    data.user?.nickname ||
    data.uniqueId ||
    data.nickname ||
    "izleyici";
}

const client = new TikTokLiveClient("sakinadilmuhammed");

client.on(EventType.follow, (data) => {
  sendEvent({
    type: "follow",
    username: getUsername(data)
  });
});

client.on(EventType.gift, (data) => {
  sendEvent({
    type: "gift",
    username: getUsername(data),
    giftName: data.gift?.name || "Hediye",
    diamonds: data.gift?.diamondCount || 0,
    count: data.repeatCount || 1
  });
});

client.on(EventType.chat, (data) => {
  sendEvent({
    type: "chat",
    username: getUsername(data),
    message: data.content || ""
  });
});

server.listen(PORT, "0.0.0.0", () => {
  console.log("Sunucu hazir: " + PORT);
});

client.connect().then(() => {
  console.log("TikTok baglantisi kuruldu!");
}).catch((error) => {
  console.error("TikTok baglanti hatasi:", error);
});
