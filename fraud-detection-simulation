import http from 'k6/http';
import { sleep } from 'k6';
import { randomIntBetween, randomItem } from 'https://jslib.k6.io/k6-utils/1.2.0/index.js';

export const options = {
  vus: 20,
  duration: '30s',
  rps: 10,
};

let userLastTxnTime = {};
let userLastDevice = {};
let userHistoricalPrices = {};

function generateTransaction(userId) {
    const productName = randomItem(['WidgetA', 'WidgetB', 'WidgetC']);
    const quantity = randomIntBetween(1, 5);
    const basePrice = randomIntBetween(10, 100);

    if (!userHistoricalPrices[userId]) userHistoricalPrices[userId] = [];

    let isFraud = Math.random() < 0.3;
    let currentPrice = basePrice;

    if (isFraud && Math.random() < 0.4) {
        let avgPrice = userHistoricalPrices[userId].length > 0
        ? userHistoricalPrices[userId].reduce((a,b) => a+b, 0) / userHistoricalPrices[userId].length
        : basePrice;
        currentPrice = avgPrice * 4;
    } else {
        currentPrice = basePrice;
    }

    let deviceName = `Device${randomIntBetween(1, 5)}`;
    const lastDevice = userLastDevice[userId];
    const now = Date.now();

    if (isFraud && Math.random() < 0.4) {
        deviceName = lastDevice ? `Device${((parseInt(lastDevice.slice(-1)) % 5) + 1)}` : deviceName;
        userLastTxnTime[userId] = now - randomIntBetween(1, 59000);
    } else {
        userLastTxnTime[userId] = now;
        userLastDevice[userId] = deviceName;
    }

    userHistoricalPrices[userId].push(currentPrice);
    if (userHistoricalPrices[userId].length > 10) {
        userHistoricalPrices[userId].shift();
    }

    const totalPrice = currentPrice * quantity;
    const status = 'PENDING';

    return {
        userId,
        productName,
        quantity,
        price: Math.floor(currentPrice),
        totalPrice,
        status,
        channel: randomItem(['WEB', 'MOBILE', 'API']),
        userIp: `192.168.${randomIntBetween(1,255)}.${randomIntBetween(1,255)}`,
        deviceName,
        location: randomItem(['Jakarta', 'Bandung', 'Surabaya']),
    };
}

export default function () {
    const userId = randomIntBetween(1, 20);
    const txn = generateTransaction(userId);

    const url = 'http://localhost:1234/graphql'; // endpoint GraphQL

    const mutation = `
        mutation CreateTransaction($request: TransactionRequest) {
        createTransaction(request: $request) {
            id
            transactionalCode
            name
            email
            productName
            quantity
            price
            totalPrice
            status
        }
        }
    `;

    const graphqlPayload = JSON.stringify({
        query: mutation,
        variables: { request: txn },
    });

    const params = {
        headers: { 'Content-Type': 'application/json' },
    };

    http.post(url, graphqlPayload, params);
    sleep(0.1);
}
