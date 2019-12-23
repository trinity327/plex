var express = require('express')
  , request = require('request')
  , multer  = require('multer');;

var app = express();
var upload = multer({ dest: '/tmp/' });

app.post('/', upload.single('thumb'), function (req, res, next) {
  var payload = JSON.parse(req.body.payload);
  var cURL = "http://localhost:8123/api/";
  var json = { "entity_id": "31971922:/zone/1" };
  var cType = "application/json";
  var ha_passwd = "(Home Assistant API 9a64b7c7-2ffb-460a-a58c-c065f899797b)";

  console.log('Got webhook for', payload.event);

  if(payload.event === 'media.play' || payload.event === 'media.resume')
  {
    console.log("Media is playing.");
    request({
      url: cURL + "services/light/turn_off",
      method: "POST", json: true, body: json,
      headers: { "x-ha-access" : ha_passwd, "Content-Type": cType }
    }, function(error, response, body) { console.log(response); });
  }
  else if(payload.event === 'media.pause' || payload.event === 'media.stop')
  {
    console.log("Media is stopped.");
    request({
      url: cURL + "services/light/turn_on",
      method: "POST", json: true, body: json,
      headers: { "x-ha-access" : ha_passwd, "Content-Type": cType }
    }, function(error, response, body) { console.log(response); });
  }
  res.sendStatus(200);
});

app.listen(32400);

console.log('Got webhook for ', payload.event + ", UUID " + payload.Player.uuid );

//  if(payload.Player.uuid == "(7.26.0.14578.)")
//  {
    if(payload.event === 'media.play' || payload.event === 'media.resume')
    {
      console.log("Media is playing.");
      request({
        url: cURL + "services/light/turn_off",
        method: "POST", json: true, body: json,
        headers: { "x-ha-access" : ha_passwd, "Content-Type": cType }
      }, function(error, response, body) { console.log(body); });
    }
    else if(payload.event === 'media.pause' || payload.event === 'media.stop')
    {
      console.log("Media is stopped.");
      request({
        url: cURL + "services/light/turn_on",
        method: "POST", json: true, body: json,
        headers: { "x-ha-access" : ha_passwd, "Content-Type": cType }
      }, function(error, response, body) { console.log(body); });
//    }
}


