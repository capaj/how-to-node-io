# Node I/O
Node.js has come a long way, so too has its asynchronous support with promises and streams. 

This guide will demonstrate some of the better practices that I have adopted while working with the file system and other I/O based operations using Node.js and the native Node modules.

# Asynchronous non-blocking File Operations
### Reading
- Reading small files / most files:
  - All files read using this method are read in-memory.
  
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
### Writing
### Streaming
