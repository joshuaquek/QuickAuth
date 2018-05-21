# QuickAuth

=========

Quick and secure asymmetric authentication, built using Node's native crypto library.

## Installation

  `npm install @jdaudier/number-formatter`

## Usage



## Full Example Usage

Say you are using Express as a wrapper for your on NodeJS server, inside `server.js`/`index.js`:

```javascript
// ---- NPM Imports ----
const QuickAuth = require('quickauth')
const bodyParser = require('body-parser')
const express = require('express')
const cors = require('cors')
const disallowCrossOrigin = cors({
  origin: "http://www.yourwebsitename.com", // This ensures that all requests can only come from this URL
  optionsSuccessStatus: 200
})



// QUICKAUTH: CONFIGURES QUICKAUTH TO USE MONGO DB AS STORAGE 
QuickAuth.configure({
  storage: 'mongodb',  // QUICKAUTH: SPECIFIES THE TYPE OF STORAGE FOR QUICKAUTH TO USE (For now its only 'mongodb' or 'lokijs')
  details: {
    databaseConnectionString: 'mongod://localhost:27017/mongoDbNameHere', // QUICKAUTH: YOUR MONGO DB CONNECTION STRING HERE
    storageTable: 'auth_keypair_collection' // QUICKAUTH: YOUR MONGO DB TABLE THAT QUICKAUTH WILL USE TO STORE PRIVATE-PUBLIC KEYPAIRS
  }
})



// ---- Express Server Setup ----
var app = express()
app.use( bodyParser.json() )
app.use( bodyParser.urlencoded( { extended: true } ) )



// ---- API Endpoints ----

// ____ 1. Front-end to first fetch public key to be used for logging in ____
app.get('/initAuth', disallowCrossOrigin, (req, res, next) => {
  let publicKey = QuickAuth.generate() // QUICKAUTH: GENERATES A RSA PUBLIC-PRIVATE KEYPAIR. RSA KEYPAIR GETS STORED IN DB WHILE PUBLIC KEY GETS RETURNED
  res.json({ public_key: publicKey }) 
})

// ____ 2. Login using the following request in json: { "public_key": "Example Public Key From FrontEnd", "payload": "Example Encrypted Payload" } ____
app.post('/login', disallowCrossOrigin, (req, res, next) => {
  // Get public key from http response
  let publicKey = (req.body || { public_key: "" }).public_key 

  // Get encrypted payload from http response
  let payload = (req.body || { payload: "" }).payload 

  // QUICKAUTH: CHECKS STORAGE USING PUBLIC KEY IF RSA PUBLIC-PRIVATE KEYPAIR EXISTS. IF SO, THEN DECRYPT THE PAYLOAD WITH PRIVATE KEY
  let data = QuickAuth.decrypt(publicKey, payload) 
  let { username, password } = data || {username: "", password: ""}

  /*
  ......
  ......
  ......
  ...... Here you can check database etc or however you want to verify if the credentials are correct...
  ...... Please note that it is NOT a good practice to store passwords in a database. 
  ...... Rather, you should store a hash, which could be a string concatenation of the username and password.
  ...... Then when you have a username and password, to verify, you can hash that and compare it with the hash in the DB to see if it matches.
  ......
  ......
  ......
  */

  if( /* Username/Password is correct */ ){

    // QUICKAUTH: GENERATE WEB TOKEN 
    let accessToken = QuickAuth.webtoken(publicKey, { identity: "Username or Identifier Here", timeoutInSeconds: 2592000 })
    // Send success response with Web Token if username and password is correct
    res.json({status: 'success', access_token: accessToken})

  }else{

    // Sends failure response
    res.json({status: 'failure', access_token: null})

  }
  
})




// ---- Start Server on 127.0.0.1:3000 ----
app.listen(3000);
```


## Tests

  `npm test`

## Special Thanks 

https://www.nodejsera.com/nodejs-tutorial-day10-crypto-module-symmetric-asymmetric-encryption-decryption.html
https://github.com/juliangruber/keypair

## Contributing

In lieu of a formal style guide, take care to maintain the existing coding style. Add unit tests for any new or changed functionality. Lint and test your code.
