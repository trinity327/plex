 lines (31 sloc) 1.14 KB
var express = require('express')
  , request = require('request')
  , multer  = require('multer');

var app = express();
var upload = multer({ dest: '/tmp/' });

app.post('/', upload.single('thumb'), function (req, res, next) {
  var payload = JSON.parse(req.body.payload);
  console.log('Got webhook for', payload.event);

  // Galaxy S10+.
  if (payload.Player.uuid == process.env.PLAYER && payload.Metadata.type != 'track') {
    var options = {
      method: 'PUT',
      json: true,
    };

    if (payload.event == 'media.play' || payload.event == 'media.resume') {
      // Turn light off.
      console.log('Turning light off.');
      options.body = { desired_state: { powered: false } };
      request(options);
    } else if (payload.event == 'media.pause' || payload.event == 'media.stop') {
      // Turn light on.
      console.log('Turning light on.');
      options.body = { desired_state: { powered: true, brightness: 1.0 } };
      request(options);
    }
  }

  res.sendStatus(200);
});

app.listen(32400);
