# Travel Memory

Assignment 2 deployment by Aditya Verma. The MERN application is deployed on two EC2 instances across Availability Zones, behind an AWS Application Load Balancer, with Cloudflare DNS and ACM HTTPS.

- Application: [travel.iadityalabs.xyz](https://travel.iadityalabs.xyz)
- [Deployment report and screenshots](docs/deployment.md)
- [Architecture preview](docs/architecture.svg) and [editable draw.io diagram](docs/architecture.drawio)
- [Nginx configuration](deploy/travelmemory.nginx.conf) and [systemd service](deploy/travelmemory-backend.service)
- [Final review and submission checklist](docs/submission-checklist.md)

Original application source from [UnpredictablePrashant/TravelMemory](https://github.com/UnpredictablePrashant/TravelMemory); the original [license](LICENSE) is retained. The instructions below describe local development. The deployment report describes production configuration on backend port **3000** with frontend API URL **`/api`**.

`.env` file to work with the backend after creating a database in mongodb: 

```
MONGO_URI='ENTER_YOUR_URL'
PORT=3001
```

Data format to be added: 

```json
{
    "tripName": "Incredible India",
    "startDateOfJourney": "19-03-2022",
    "endDateOfJourney": "27-03-2022",
    "nameOfHotels":"Hotel Namaste, Backpackers Club",
    "placesVisited":"Delhi, Kolkata, Chennai, Mumbai",
    "totalCost": 800000,
    "tripType": "leisure",
    "experience": "Lorem Ipsum, Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum, ",
    "image": "https://t3.ftcdn.net/jpg/03/04/85/26/360_F_304852693_nSOn9KvUgafgvZ6wM0CNaULYUa7xXBkA.jpg",
    "shortDescription":"India is a wonderful country with rich culture and good people.",
    "featured": true
}
```


For frontend, you need to create `.env` file and put the following content (remember to change it based on your requirements):
```bash
REACT_APP_BACKEND_URL=http://localhost:3001
```

## How to run BE

Note: Make sure you have the .env file already added
```bash
cd backend
npm install
node index.js
```

## How to run FE
Note: Make sure you have the .env file already added
```bash
cd frontend
npm install
npm start
```
