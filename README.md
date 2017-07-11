# Zoetis Find-A-Vet Sample

A project that builds a VueJS app and component to provide geospatial search over veterinary practices. 

The whole solution is composed of two parts:

* Practice Data Management: this is currently handles via the Crownpeak CMS that pushes geocoded practice data to a Crownpeak Search G2 collection
* Front-end: this project -- implements a user interface to allow web visitors to pick a search range, say 5 miles, and enter their address information, typically just a postal code, and then search the collection for matching practices.

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

