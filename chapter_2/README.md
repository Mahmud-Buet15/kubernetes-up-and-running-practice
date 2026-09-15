## 01_simple_node
- **package.json**: This is the configuration file for a Node.js project
- **server.js**: This is the actual application code 
- To run the app in local machine
    - Install node: `brew install node`  
        This will also install **npm** (Node Package Manager) which you'll need for managing Node.js packages
    - Install dependencies (here need to install express) : `npm install express`
    - Run the application: `npm start`
    - Test the application:
        - From command line: `curl http://localhost:3000`
        - From browser: `http://localhost:3000`
- To dockerize the app
    - Add dockerfile and .dockerignore file. Add necessary commands 
    - Build docker image: `docker build -t simple-node .`
    - Run a container from the image: `docker run --rm -p 3000:3000 simple-node`
