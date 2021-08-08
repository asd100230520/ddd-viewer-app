# DDD Viewer Web App

DDD Viewer Web App is a web application for visualizing 3D (geographical) data, generated by the
[ddd](https://github.com/jjmontesl/ddd) project.

## DDD Viewer Online

You can find DDD Viewer online at the 3dsmaps.com site, which uses 

- [Trip around A Coruña - Spain](https://3dsmaps.com/play/?u=/trips/acoruna-menhires.json&sb=@dynamic&@43.3861094,-8.4069793,84a,35y,127.1h,94.38t)
- [Trip around Vigo - Spain](https://3dsmaps.com/play/?u=/trips/vigo-castro.json&@42.2317848,-8.7279656,131a,35y,28.9h,51.29t)


## Development

Clone the repository:

    git clone core.autocrlf=false https://github.com/jjmontesl/ddd-viewer-app.git

### Compiles and hot-reloads for development

    npm run serve

or

    npm run serve-pub-https

The build toolchain accepts some environment variables that configure the application:

    DDD_VIEWER_APP_TILE_URL_BASE="http://localhost:8000/cache/ddd_http/" npm run serve

### Assets

Note that this application connects to map tile servers to retrieve and show data. It also requires
some locally available assets to be included with the application. Please refer to 
[ddd-viewer](https://github.com/jjmontesl/ddd-viewer) project documentation for further information.

TODO: how to download assets to be used locally (document here or in ddd-viewer project).


### HTML5 History Mode considerations

This project uses Vue Router HTML5 History Mode, this means when you are in development mode you can hit Cmd+R (mac) or F5 (Windows) to reload the page and it will work, but when the project is built you will have problems, so will need to do a small change in your web server to make that work. Please read the official Vue Router documentation here: <https://router.vuejs.org/guide/essentials/history-mode.html#example-server-configurations>


### Compiles and minifies for production

```bash
npm run build
```

## Bugs or improvements

Feel free to report any bugs or improvements. Pull requests are always welcome.


## License

Copyright (c) 2021 Jose Juan Montes and contributors

See LICENSE file for further information.



