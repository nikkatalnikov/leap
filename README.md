![alt text](https://github.com/nikkatalnikov/leap/raw/master/media/logo.png "LEAP.JS")

###**LEAP.JS - reactive bindings for IO-actions and more.** 
Leap is a tiny library written in ES6 with RxJS to provide concise and robust infrastructure for driving **data layer abstractions**: **HTTP**, **SSE**, **WS**, other IO-actions and strictly evaluated data types **as multidirectional reactive streams** (ie. binding IO and Observable/Observer with Rx.Subject and vice versa).

[![Build Status](https://img.shields.io/travis/nikkatalnikov/leap/master.svg?style=flat-square)](https://travis-ci.org/nikkatalnikov/leap)
[![Code Climate](https://img.shields.io/codeclimate/github/nikkatalnikov/leap.svg?style=flat-square)](https://codeclimate.com/github/nikkatalnikov/leap)
[![Latest Stable Version](https://img.shields.io/npm/v/leap-js.svg?style=flat-square)](https://www.npmjs.com/package/leap-js)
[![Dependency Status](https://img.shields.io/david/nikkatalnikov/leap.svg?style=flat-square)](https://david-dm.org/nikkatalnikov/leap)
[![devDependency Status](https://img.shields.io/david/dev/nikkatalnikov/leap.svg?style=flat-square)](https://david-dm.org/nikkatalnikov/leap#info=devDependencies)
[![NPM Downloads](https://img.shields.io/npm/dm/leap-js.svg?style=flat-square)](https://www.npmjs.com/package/leap-js)

####**Install**
NPM:

	npm i leap-js -S

then hook up Leap.js into project:

ES6:

	import { StreamAPI } from 'leap-js'

Node / Browserify:

	const StreamAPI = require('leap-js').StreamAPI

UMD:

	<script src="leap/dist/leap.min.js"></script>

####**API**
Import Leap:

	import { StreamAPI } from 'leap-js'

Create streamer instance with following data structures:

	StreamAPI :: TYPE -> OPTIONS -> Streamer
	
	-- JS: const Streamer = new StreamAPI(TYPE, OPTIONS);
	-- notice that args are not curried.

	data TYPE = "HTTP" | "WS" | "SSE"

	data OPTIONS = 
		HTTPOptions {
			credentials :: Credentials, 
			config :: AxiosConfig, 
			endpoints :: LeapEndpoints
		} | 
		WSSSEOptions {
			endpoint :: Url,
			withCredentials: Bool
		}

API:

	-- consider all above as curried with partially applied Streamer 
	-- JS: Streamer.dataStream ...

	-- common
	dataStream :: Observable a
	errorStream :: Observable a

	-- for HTTP and WS
	send :: data -> IO ()
	sendMany :: [data] -> Maybe delay -> IO ()
	
	-- for SSE and WS
	close :: IO ()

	
	-- Group API - HTTP only
	-- creates new Streamer instance with the endpoints matched by:
	-- name (multiple args) / url (single arg) / method (single arg)

	-- JS: Streamer.groupByName('ep1','ep2',...'epN')
	groupByName :: [ep] -> Streamer
	groupByName [ep] = StreamAPI "HTTP" HTTPOptions { endpoints :: matchedEps, ... }
			where matchedEps = filter pred (endpoints Streamer)

	groupByUrl :: Url -> Streamer
	groupByMethod :: Method -> Streamer

####**Examples HTTP**
Prepare config (for config details check [AXIOS API](https://github.com/mzabriskie/axios#axios-api "AXIOS API")):

	const credentials = {};
	const config = {
	  baseURL: 'http://localhost:3000'
	};

Add endpoints declaratively:

	const endpoints = {
	  removePost: {
	    url: '/posts/:id',
	    method: 'delete'
	  },
	  addPost: {
	    url: '/posts',
	    method: 'post'
	  },
	  getPosts: {
	    url: '/posts',
	    method: 'get'
	  },
	  getPost: {
	    url: '/posts/:id',
	    method: 'get'
	  },
	  editPost: {
	    url: '/posts/:id',
	    method: 'put'
	  }
	}

Create Leap instance:

	const StreamAPI = require('leap-js').StreamAPI;
	const DL = new StreamAPI('HTTP', { endpoints, config, credentials });

Run REST API server and add subscription:

	DL.dataStream.subscribe(res => {
	  console.log('Data Stream:');
	  console.log(res.data);
	});

	DL.dataStream.subscribe(insertDataInDOM);
	
	DL.errorStream.subscribe(err => {
	  console.log('Error Stream:');
	  console.error(err);
	});

	DL.errorStream.subscribe(err => {
	  if (err.statusText) {
	    $('#notifications').text(err.statusText);
	  } else {
	    $('#notifications').text(err);
	  }
	});

	DL.errorStream.subscribe(err => {
	  console.log('SSE Error Stream:');
	  console.error(err);
	});


Senders example:
	
	DL.send('getPosts');
	DL.send('getPost', {id:60});
	DL.sendMany([['getPosts', {id: 1}],['getPosts', {id: 2}]]);


####**Examples WS/SSE**
Create Leap instance:

	const StreamAPI = require('leap-js').StreamAPI;
	const DLWS = new StreamAPI('WS', 'ws://localhost:3001');

Run server and add subscription:

	DLWS.dataStream
		.take(10)
		.do(() => void 0, () => void 0, () => {
			DLWS.close();
			console.log('DLWS stopped');
		})
		.subscribe((res) => {
			console.log('WS Data Stream:');
			res.type === 'message' ?
			  console.log(res.data) :
			  console.log(res.type);
		});

	DLWS.errorStream
		.subscribe(err => {
		  console.log('SSE Error Stream:');
		  console.error(err);
		});


Senders example:
	
	DLWS.sendMany([{data:'1'},{data:'2'},{data:'3'}], 1000);

	setTimeout(() => {
	  DLWS.send({data:'x'});
	}, 3000)

Same way works for SSE.

Check more exmaples in /exmaples folder

####**License**
ISC

####**TODO (Beta-1 release)**

CLIENT (ClientAPI)

	1. HTTPS check
	2. HTTP query params
	3. SSE with credentials check
	4. WSS check
	5. WS/SSE reconnect: Bool
	6. WS/SSE buffer messages until opened
	7. SSE onclose

####**TODO (Beta-2 release)**

CLIENT (ClientAPI)

	1. HTTP.poll
	2. HTTP.pollUntil
	3. Notification API

SERVER (ServerAPI)

	1. SSE custom
	2. WS ws
	3. Regis redis
	4. Mongo mongodb


####**What's next**
1. Notification API
2. NodeJS express integration: to be discussed
3. NodeJS libs/orms integration: ws, redis, mongoose, SSE (custom)
4. List a -> ObservableCollection (List a)
5. Object {a} -> MutationObserver (Object {a})

####**Future exmaples**
1. DL -> Controller -> Stateless Components (React)
2. DL -> Stateless Services -> VM (Angular 1.5)
3. Architecture guide: async MVC (DL -> Controller -> View)
