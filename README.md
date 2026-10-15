# ShopKart

A full-stack e-commerce app built with the MERN stack: register and log in (JWT in an HttpOnly cookie),
browse and search products, keep a wishlist and a cart, pay with Razorpay (test mode) and view your orders.

- **Backend:** Node.js, Express 5, MongoDB (Mongoose), bcrypt, jsonwebtoken, cookie-parser, Razorpay
- **Frontend:** React 19, Vite, React Router, Axios

## Setup

Requires Node.js 20.6 or newer, a MongoDB Atlas database and a Razorpay account in test mode.

### 1. Backend

```sh
cd backend
npm install
copy .env.example .env      # Windows (use cp on Mac/Linux), then fill in the values
npm run seed                # adds 12 sample products
npm run dev
```

`backend/.env`:

```dotenv
PORT=3000
MONGO_URI=mongodb+srv://USER:PASSWORD@cluster0.xxxxx.mongodb.net/shopkart
JWT_SECRET=a_long_random_string
NODE_ENV=development
CLIENT_URL=http://localhost:5173
RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxx
RAZORPAY_KEY_SECRET=xxxxxxxxxxxxxxxx
```

Get the Razorpay keys from dashboard.razorpay.com: switch to **Test Mode**, then
**Account & Settings -> API Keys -> Generate Test Key**.

### 2. Frontend (second terminal)

```sh
cd shopkart-frontend
npm install
npm run dev
```

Open http://localhost:5173 and register an account.

## Test payments

At checkout use the UPI ID `success@razorpay`, or pick Netbanking / Wallet and choose "Success" in the test popup.

## API

All routes except register, login and the product list/detail need the auth cookie.

| Area      | Method | Endpoint                       | Description                               |
| --------- | ------ | ------------------------------ | ----------------------------------------- |
| Customers | POST   | `/customers/register`          | Register                                  |
| Customers | POST   | `/customers/login`             | Log in (sets the cookie)                  |
| Customers | GET    | `/customers/me`                | Current customer                          |
| Customers | POST   | `/customers/logout`            | Log out (clears the cookie)               |
| Customers | PATCH  | `/customers/change-password`   | Change password                           |
| Products  | GET    | `/products`                    | List (`?search=&category=&sort=price_asc`) |
| Products  | GET    | `/products/:id`                | One product                               |
| Products  | POST   | `/products`                    | Create a product (logged in)              |
| Wishlist  | GET    | `/wishlist`                    | My wishlist                               |
| Wishlist  | POST   | `/wishlist/:productId`         | Add                                       |
| Wishlist  | DELETE | `/wishlist/:productId`         | Remove                                    |
| Cart      | GET    | `/cart`                        | My cart                                   |
| Cart      | POST   | `/cart/:productId`             | Add one unit                              |
| Cart      | PATCH  | `/cart/:productId`             | Set quantity `{ "quantity": 3 }`          |
| Cart      | DELETE | `/cart/:productId`             | Remove                                    |
| Orders    | POST   | `/orders/create-payment-order` | Create a Razorpay order from the cart     |
| Orders    | POST   | `/orders/verify-payment`       | Verify the payment signature              |
| Orders    | GET    | `/orders`                      | My paid orders                            |
| Orders    | GET    | `/orders/:id`                  | One order                                 |

## How checkout stays secure

1. The server builds the order from the **server-side cart**, so the browser can never pick the price.
2. Razorpay takes the payment in its own popup.
3. The browser sends the payment result back and the server **recomputes the HMAC signature** with the secret key.
   Only a valid signature marks the order PAID, reduces stock and clears the cart.

## Project structure

```text
backend/
  controllers/  models/  routes/  middlewares/  utils/
  index.js  seed.js
shopkart-frontend/
  src/
    components/  context/  pages/  services/
```
