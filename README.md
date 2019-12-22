var express = require('express')
  , request = require('request')
  , multer  = require('multer');

var app = express();
var upload = multer({ dest: '/tmp/' });

app.post('/', upload.single('thumb'), function (req, res, next) {
  var payload = JSON.parse(req.body.payload);
  console.log('Got webhook for', payload.event);

  // Firefox.

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

{"sessionId":"8d4dea94-4d9b-485c-a97a-867f71b4106a","requestId":"2d17f965-5b12-4189-8d67-9094e6b09131","responseText":"Done.","responseCode":"SESSION_SUCCESS","followUpExpected":false,"sessionComplete":true,"json":null}
