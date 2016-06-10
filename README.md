# Node I/O
Node.js has come a long way, so too has its asynchronous support with promises and streams. 

This guide will demonstrate some of the better practices that I have adopted while working with the file system and other I/O based operations using Node.js and the native Node modules.

# Asynchronous non-blocking File Operations
#### Reading
- Reading small files / most files:
  - All files read using this method are read in-memory.
  - The largest file size you can read here is roughly 2gb, anything bigger will throw this error: `RangeError: File size is greater than possible Buffer: 0x7fffffff bytes at FSReqWrap.readFileAfterStat [as oncomplete] (fs.js:386:11)`
  - Because this is read in memory - it is not the optimal solution for files exceeding hundreds of megabytes, please see method below this one.
  ```
  // Modern approach
  function readFile(fileName, options) {
      return new Promise(function(resolve, reject) {
      	var cb = function cb(err, data) {
      		if (err) {
      			return reject(err);
      		}
  
      		resolve(data);
      	};
  
      	fs.readFile.call(fs, fileName, options, cb);
      });
  }

  
  // example:
  readFile(__dirname + "/file.txt").then(function(data) {
     // data = the whole file
	   console.log(data.length);
  }).catch(function(err) {
  	console.log(err);
  })
  ```
  
- Reading larger files
  - All files read using this method can read any sized file, since it streams a number of bytes in at a time
  - We set `highWaterMark` to be equal to the size of the buffer we want, roughly 512kb of data stored in memory (very efficient)
  - The callback will have the data in chunks of ~512kb at a time (taking in any more bytes doesnt really help performance)
  - The example demonstrates reading a 7gb file in roughly 5minutes (212 seconds) (https://dumps.wikimedia.org/other/wikibase/wikidatawiki/ - latest-all.json.gz)
  - Note: the data here is gzipped and will not be converted to readable format, for unzipping and streaming readable data see the method below this one.
  ```
  // Modern approach
  function readFileStream(fileName, cb, options) {
	    return new Promise(function(resolve, reject) {
	        var readable = fs.createReadStream(fileName, options);
	
	        readable.on('data', function(data) {
	            cb(data);
	        })
	
	        readable.on('end', function() {
	            resolve();
	        })
	
	        readable.on('error', function(err) {
	            return reject(err);
	        })
	    })
  }
  
  // example:
  function printDataAsItComesIn(data) {
    // Data will be null when it reaches the end.
    if (!data) return;
    
	// type of data = Buffer
	var stringData = data.toString('utf8', 0, data.length);
	console.log(stringData.length);
  }
  
  readFileStream(__dirname + "/latest-all.json.gz", printDataAsItComesIn, {highWaterMark: 512 * 1024}).then(function() {
	console.log("\nDone reading");
  }).catch(function(err) {
	console.log(err);
  })
  ```
- Reading and unzipping larger files (7gb file fize)
  ```
  function unzipFileStream(fileName, cb, options) {
    return new Promise(function(resolve, reject) {
        var readable = fs.createReadStream(fileName, options);
        var unzip = zlib.createGunzip({chunkSize: 512 * 1024})
        readable.pipe(unzip)

        unzip.on('data', function(data) {
            cb(data);
        })

        unzip.on('end', function() {
            resolve();
        })

        unzip.on('error', function(err) {
            return reject(err);
        })
    })
  }

  function printDataAsItComesIn(data) {
    if (!data) return;
	// data = Buffer
	
	// here we can actually read the data in the zip file
	var stringData = data.toString('utf8', 0, data.length);
	console.log(stringData);
  }

  unzipFileStream(__dirname + "/latest-all.json.gzip", printDataAsItComesIn, {highWaterMark: 512 * 1024}).then(function() {
	console.log("\nDone reading");
  }).catch(function(err) {
	console.log(err);
  })
  ```
