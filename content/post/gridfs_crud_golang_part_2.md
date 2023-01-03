---
title: "GridFS with GoLang (Part 2)"
date: 2022-12-10T17:50:44+05:30
tags: ['golang', 'gridfs', 'mongodb']
# draft: true
---

<span style="font-size:25px; font-family:'Kalam'">
<br> Hello Again!

If this is your first encounter with my "story telling" skill :P, I'll recommend checking out the [Part 1](https://apoorvajagtap.github.io/gridfs_crud_golang) of "GridFS with GoLang" series first.

This series focuses on GridFS implementation with goLang, hence we'll be skipping the route Handler details, and how mongodb setup was initialized. <br>Here, I'll be sharing the functions to perform GET, POST, UPDATE/PUT and DELETE operations on the files stored in the server.

I am skipping the aforementioned topics, as we have a several useful+detailed blogs which help with these configurations. However, if any of you is interested to have the complete code discussed, feel free to ping me via my social media accounts ([About me](https://apoorvajagtap.github.io/about/)), and I'll be happy to share that as well!

We are using [gridfs](https://pkg.go.dev/go.mongodb.org/mongo-driver/mongo/gridfs) package, which provides a MongoDB GridFS API.
<br><br>

➤ **[POST] Upload the file**

As soon as our server starts, it creates a client instance to establish a connection with the mongoDB server, which is running as a container. 

Sharing some essential snippets w.r.t same. Complete code is available within [dbase](https://github.com/apoorvajagtap/fileStore/blob/master/dbase/config.go) & [server](https://github.com/apoorvajagtap/fileStore/blob/master/server/server.go).

```go
//client instance
var DB *mongo.Client = ConnectDB()

// connect to the DB
func ConnectDB() *mongo.Client {
	client, err := mongo.NewClient(options.Client().ApplyURI(mongoURI))
    ...
}
```

Once the connection to mongoDB is established, the server starts to listen & serve at port `4500`, and any incoming request is managed by the specific route handlers.  

The `uploadFileHandler` undertakes the following:
1. Using [ParseMultipartForm](https://pkg.go.dev/net/http#Request.ParseMultipartForm), parses the incoming request body as multipart/form-data upto 32Mb bytes stored in memory.
2. Creates a GridFS bucket The `bucket` will basically wrap the mongo.Database instance and will operate on the two collections that'll be used for storing the file's content (fs.chunks) & file's metadata (fs.files).
3. The server allows uploading multiple files in one go, so, looping through all the files coming with the http request.
4. Check for an existing file with the same name. Error out if a POST request is made for an existing file.
5. Reading through the file, and copying the file content to a buffer named `buff`.
6. Creates a new uploadStream for the upcoming file using [OpenUploadStream()](https://github.com/mongodb/mongo-go-driver/blob/master/mongo/gridfs/bucket.go#L116), and then [Write()](https://github.com/mongodb/mongo-go-driver/blob/master/mongo/gridfs/upload_stream.go#L95) function writes the byte slice's content to the respective uploadstream.

```go
func uploadFileHandler(w http.ResponseWriter, r *http.Request) {

    // Checks the http method. Allows only POST (upload new file) & PUT (edit existing file content)
	if r.Method != "POST" && r.Method != "PUT" {
		http.Error(w, "Method Not Allowed!", http.StatusMethodNotAllowed)
		return
	}
	_, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	// 1. Parsing the request body as multipart/form-data upto 32Mb bytes stored in memory
	err := r.ParseMultipartForm(32 << 20)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	files := r.MultipartForm.File["file"]
	// 2. Creates a bucket
	bucket, err := gridfs.NewBucket(
		fileCollection.Database(),
	)
	if err != nil {
		log.Fatal(err)
		os.Exit(1)
	}
    // 3. Loop through all the files sent with the request.
	for _, fileHeader := range files {
		file, err := fileHeader.Open()
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}
		defer file.Close()
		// 4. Checks if file with same name already exists
		if r.Method == "POST" && fileExists(fileHeader) {
			http.Error(w, fmt.Sprintf(">>> The file '%s' already exists!\n", fileHeader.Filename), http.StatusInternalServerError)
			log.Printf("The file '%s' already exists!\n", fileHeader.Filename)
			continue
		}
	
		// 5. Saving the file data in buffer
		buff := bytes.NewBuffer(nil)
		if _, err := io.Copy(buff, file); err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
		}
		// 6. Creates a file ID new uploadstream for the upcoming file, and transfer the content of byte slice to the specific uploadStream.
		uploadStream, err := bucket.OpenUploadStream(fileHeader.Filename, uploadOpts)
		if err != nil {
			fmt.Println(err)
			os.Exit(1)
		}
		defer uploadStream.Close()
		fileSize, err := uploadStream.Write([]byte(buff.String()))
		if err != nil {
			log.Fatal(err)
			os.Exit(1)
		}
		// Print the status
		log.Printf("Write file to DB was successful. File size: %d\n", fileSize)
		fmt.Fprintf(w, ">>> File %s uploaded successfully", uploadStream.FileID)
	}
}
```

Now, time for a break! 

We'll continue our discussion about rest of the http Methods in Part 3 of the same series (GridFs with GoLang). See you there :D 
</span>