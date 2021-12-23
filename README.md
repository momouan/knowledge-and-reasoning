# A basic and simplistic version of "A Good Snowman Is Hard To Build". The front-end was developed using Vue.js, and the logic is managed by an ontology loaded on Stardog and in which we make queries using SPARQL.

The idea here is to minimize the use of conditionals (if..else, switch case) and instead have the logic managed by reasoning on the ontology.
The ontology was created on Protege-5.5.0 and saved as a turtle file which is therefore loaded in Stardog server 7.7.2 mounted as a Docked container. And finally, The front-end has been developed with Vue.js 3.

# Download the container
```
sudo docker pull stardog/stardog:latest
```

# Start the container
```
sudo docker run -it -v ~/stardog-home/:/var/opt/stardog -p 5820:5820 stardog/stardog
```

# Install the stardog package for nodejs
```
npm install stardog
```

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

