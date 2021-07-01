# client
I had trouble finding any good information on uploading images for a whatsapp style chat application that used web sockets. Socket io says they can do it but the documentation seems not to be available. To get round this, the client side sends the image to the server via axios to be parsed by multer, which saves the file on the client side and emits the image url via socket io.
If you have another solution feel free to pull and create a new branch!
The node/express server uses multer to read files and can be found [here](https://github.com/al-mister-dog/chat-app)
## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
