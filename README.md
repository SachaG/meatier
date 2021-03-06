# meatier

Meteor is awesome! But after 3 years, it's starting to show it's age. This project is designed to showcase 
the exact same functionality as Meteor, *but without the monolithic structure.* 
It trades a little simplicity for a lot of flexibility.

Some of my chief complaints with Meteor
 - Built on Node 0.10, and that ain't changing anytime soon
 - Build system doesn't allow for code splitting (the opposite, in fact)
 - Locks you in to a walled garden (npm require only gets you so far)
 - Global scope (namespacing doesn't count)
 - Goes Oprah-Christmas-special with websockets (not every person/page needs one)
 - Can't handle css-modules (CSS is all handled behind the scenes)
 - Dependent on MDG to release Meteor-specific package updates like react
 - Tied to MongoDB for official support
 
| Problem           | Meteor's solution            | My solution    | Result                                                              |
|-------------------|------------------------------|----------------|---------------------------------------------------------------------|
| Database          | [MongoDB](https://www.mongodb.org/)                      | [RethinkDB](https://www.rethinkdb.com/)      | Built in reactivity, but you can use any DB you like                |
| Database schema   | [Simple Schema](https://github.com/aldeed/meteor-simple-schema)                | [Joi](https://github.com/hapijs/joi)            | Thinky works too, but Joi has nicer error handling                  |
| Database hooks    | [Collections2](https://github.com/aldeed/meteor-collection2)                 | [Thinky](https://github.com/neumino/thinky)         | I don't use thinky as an ORM, more like a wrapper for rethinkdbdash |
| Forms             | [AutoForm](https://github.com/aldeed/meteor-autoform)                     | [redux-form](https://github.com/erikras/redux-form)     | state tracking awesomeness that works beautifully with react        |
| Client-side cache | [Minimongo](https://www.meteor.com/mini-databases)                    | [redux](http://redux.js.org/)          | Bonus logging, time traveling, and undo functionality               |
| socket server     | [DDP-server](https://www.meteor.com/ddp)                   | [socketcluster](http://socketcluster.io/#!/)  | super easy scaling, pubsub, and auth                                |
| Authentication    | Meteor accounts              | [JWTs](https://jwt.io)           | JWTs can also serve to authorize actions, too                       |
| Auth-transport    | [DDP](https://www.meteor.com/ddp)                          | HTTP           | Don't use sockets until you need to                                 |
| Front-end         | [Blaze](https://www.meteor.com/blaze)                        | [React](https://facebook.github.io/react/)          | Not dependent on MDG releases                                       |
| Build system      | meteor                       | [webpack](https://webpack.github.io/)        | using webpack inside meteor is very limited                         |
| CSS               | magically bundle & serve     | [css-modules](https://github.com/css-modules/css-modules)    | component-scoped css with variables available in a file or embedded |
| Optimistic UI     | latency compensation         | [redux-optimist](https://github.com/ForbesLindesay/redux-optimist) | super flexible!                                                     |
| Testing           | Velocity (or nothing at all) | [Ava](https://github.com/sindresorhus/ava)            | awesome es2016 testing                                              |
| Linting           | Your choice                  | [xo](https://www.npmjs.com/package/xo)             | no dotfiles, fixes errors                                           |
| Server            | Node 0.10.41                 | Node 5         | Faster, maintained, not a dinosaur...                               |
 
##Installation
- `brew install rethinkdb`
- `git clone` this repo
- `cd meatier`
- `npm install`
- `npm run build`
- `npm i -g webpack@2.0.1-beta` (optional, but recommended)
- `rethinkdb`

##Client-side development
- `npm start` (in a second terminal window)
- http://localhost:3000

Rebuilds the client code in-memory & uses hot module reload so you can develop super fast!
On my 2013 MBA an initial build takes about 8 seconds and updates usually take 800ms

##Server-side development
- `npm run prod` (in a second terminal window)
- http://localhost:3000
- If you edit any client or universal files, run `npm run bs` to rebuild & serve the bundle

This mode is great because you can make changes to the server ***without having to recompile the client code***
That means you only wait for the server to restart! GAME CHANGER!

##Webpack configs
####Development config
When the page is opened, a basic HTML layout is sent to the client along with a stringified redux store and a request for the common chunk of the JS.
The client then injects the redux store & router to create the page.
The redux devtools & logger are also loaded so you track your every state-changing action. 
The routes are loaded async, check your networks tab in chrome devtools and you'll see funny js files load now & again. 
If this isn't crazy amazing to you, then go away.

####Production config
Builds the website & saves it to the `build` folder.
Maps the styles to the components, but uses the prerendered CSS from the server config (below)
Separates the `vendor` packages and the `app` packages for a super quick, cachable second visit.
Creates a webpack manifest to enable longterm caching (eg can push new vendor.js without pushing a new app.js)
Optimizes the number of chunks, sometimes it's better to have the modules of 2 routes in the same chunk if they're small

####server config
A webpack config builds the entire contents of the routes on the server side.
This is required because node doesn't know how to require `.css`.
When a request is sent to the server, react-router matches the url to the correct route & sends it to the client.
Any browser dependency is ignored & uglified away.
To test this, disable javascript in the browser. You'll see the site & css loads without a FOUC.

##How it works
When the page loads, it checks your localStorage for `Meatier.token` & will automatically log you in if the token is legit. 
If not, just head to the 'Sign up' page. The 'Sign up' page uses redux-form, which handles all errors, schema validation,
and submissions. Your credentials are sent to a REST API and a user document (similar to Meteor's) is returned to your state.

The 'Kanban' app requires a login & websocket, so when you enter, your token will be used to authenticate a websocket.
That token is stored on the server so it is only sent during the handshake (very similar to DDP). Socket state is managed
by `redux-socket-cluster`, just clicking `socket` in the devtools let's you explore its current state. 
To make this happen, the package uses a fork of socketcluster-client (v4.0 is still in the works). 

When the component loads, it subscribes to `lanes` & `notes`, which starts your personalized changefeed.
When you do something that changes the persisted state (eg add a kanban lane) that action is executed
optimistically on the client & emitted to the server where it is validated & sent to the database. 
The database then emits a changefeed doc to all subscribed viewers.
Since the DB doesn't know which client made the mutation, it always sends a changefeed to the server.
The server is smart enough to ignore sending that document back to the originator, but it does send an acknowledgement.

The kanban lane titles & notes are really basic, you click them & they turn into input fields. 
The notes can be dragged from lane to lane. This is to showcase a local state change that doesn't affect the persisted state.
When the note is dropped to its new location, the change is persisted. 


##Similar Projects
 - https://github.com/erikras/react-redux-universal-hot-example (Really nice, but no auth or DB)
 - https://github.com/kriasoft/react-starter-kit (nice, I borrowed their layout, but no sockets, no DB)
 - https://github.com/GordyD/3ree (uses RethinkDB, but no optimistic UI)
 - http://survivejs.com/ (A nice alt-flux & react tutorial for a kanban)

##In Action
I don't know of any place that hosts RethinkDB for free...so here's a gif. 
![Meatier](http://imgur.com/B3IErZr.gif)

##License
MIT




