---
{
  "name": "Router Configuration",
  "culture": "en-US",
  "description": "This article covers Aurelia's router configuration.",
  "engines" : { "aurelia-doc" : "^1.0.0" },
  "author": {
    "name": "Jeremy Gonzalez",
    "url": "http://www.jeremyg.net"
  },
  "contributors": [],
  "translators": [],
  "keywords": ["JavaScript", "Router"]
}
---
## [Basic Configuration](aurelia-doc://section/1/version/1.0.0)

To use Aurelia's router, your component view must have a `<router-view></router-view>` element. In order to configure the router, the component's view-model requires a `configureRouter()` function.

<code-listing heading="app.html">
  <source-code lang="HTML">
    <template>
      <router-view></router-view>
    </template>
  </source-code>
</code-listing>

<code-listing heading="Basic Route Configuration">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',      name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',      name: 'files',      moduleId: 'files/index',   href:'#files',   nav: 0 }
        ]);
      }
    }
  </source-code>
</code-listing>

You can also call `mapRoute()` on a single route configuration.

* `config.map()` adds route(s) to the router.  Although only route, name, moduleId, href and nav are shown above there are other properties that can be included in a route. The class name for each route is `RouteConfig`.
* `route` - is the pattern to match against incoming URL fragments. It can be a string or array of strings. The route can contain parameterized routes or wildcards as well.
  * Parameterized routes match against a string with a `:token` parameter (ie: 'users/:id/detail'). An object with the token parameter's name is set as property and passed as a parameter to the route view-model's `activate()` function.
  * A parameter can be made optional by appending a question mark `:token?` (ie: `users/:id?/detail` would match both `users/3/detail` and `users/detail`). When an optional parameter is missing from the url, the property passed to `activate()` is `undefined`.
  * Wildcard routes are used to match the "rest" of a path (ie: files/*path matches files/new/doc or files/temp). An object with the rest of the URL after the segment is set as the `path` property and passed as a parameter to `activate()` as well.
* `href` - is a conditionally optional property. If it is not defined then route is used. If route has segments then href is required as in the case of files because the router does not know how to fill out the parameterized portions of the pattern.
* `nav` - is a boolean or number property. When set to true the route will be included in the router's navigation model. This makes it easier to create a dynamic menu or similar elements. When specified as number, the value will be used in sorting the routes.

## [Options](aurelia-doc://section/2/version/1.0.0)

### Push State

Add [a base tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base) to the head of your html document. If you're using JSPM, RequireJS or a similar module loader, you will also need to configure it with a base url, corresponding to your base tag's `href`. Finally, be sure to set the `config.options.root` to match your base tag's setting.

<code-listing heading="Push State">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config) {
        config.title = 'Aurelia';
        config.options.pushState = true;
        config.options.root = '/';
        config.map([
          { route: ['welcome'],    name: 'welcome',     moduleId: 'welcome',      nav: true, title:'Welcome' },
          { route: 'flickr',       name: 'flickr',      moduleId: 'flickr',       nav: true },
          { route: 'child-router', name: 'childRouter', moduleId: 'child-router', nav: true, title:'Child Router' },
          { route: '',             redirect: 'welcome' }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {Redirect, NavigationInstruction, RouterConfiguration} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration): void {
        config.title = 'Aurelia';
        config.options.pushState = true;
        config.options.root = '/';
        config.map([
          { route: ['welcome'],    name: 'welcome',     moduleId: 'welcome',      nav: true, title:'Welcome' },
          { route: 'flickr',       name: 'flickr',      moduleId: 'flickr',       nav: true },
          { route: 'child-router', name: 'childRouter', moduleId: 'child-router', nav: true, title:'Child Router' },
          { route: '',             redirect: 'welcome' }
        ]);
      }
    }
  </source-code>
</code-listing>

> Warning
> PushState requires server-side support. Don't forget to configure your server appropriately.

## [Dynamically Specify Route Components](aurelia-doc://section/3/version/1.0.0)

You can add a `navigationStrategy` to a route to allow dynamic routes. Within the navigation strategy Aurelia requires you to configure `instruction.config` with the desired `moduleId`, viewPorts or redirect.

<code-listing heading="Using a Route Navigation Strategy">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        this.router = router;
        config.title = 'Aurelia';
        var navStrat = (instruction) => {
          instruction.config.moduleId = instruction.fragment
          instruction.config.href = instruction.fragment
        }
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true },
          { route: ['', 'admin*path'],   name: 'route',  navigationStrategy: navStrat }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router, NavigationInstruction} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        this.router = router;
        config.title = 'Aurelia';
        let navStrat = (instruction: NavigationInstruction) => {
          instruction.config.moduleId = instruction.fragment
          instruction.config.href = instruction.fragment
        }
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true },
          { route: ['', 'admin*path'],   name: 'route',  navigationStrategy: navStrat }
        ]);
      }
    }
  </source-code>
</code-listing>

## [Adding Additional Data To A Route](aurelia-doc://section/4/version/1.0.0)

Although Aurelia does allow you to pass any additional property to a route's configuration object, `settings` is the default parameter to which you should add arbitrary data that you want to pass to the route.

<code-listing heading="Using Route Settings">
  <source-code lang="ES 2015/2016">

    export class App {
      configureRouter(config, router) {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true, settings: {data: '...'} }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true, settings: {data: '...'} }
        ]);
      }
    }
  </source-code>
</code-listing>

## [Case Sensitive Routes](aurelia-doc://section/5/version/1.0.0)

You can set a route to be case sensitive, should you wish:

<code-listing heading="Making a Route Case-Sensitive">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true, caseSensitive: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index', nav: true, caseSensitive: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' }
        ]);
      }
    }
  </source-code>
</code-listing>

In the above example, our route will only match URL fragment of '/users' and not '/Users', *but* since the route 'users/:id/detail' is not case sensitive the URL Users/:id/detail would match. By default Aurelia's routes are not case sensitive.

## [Handling Unknown Routes](aurelia-doc://section/6/version/1.0.0)

Aurelia allows you to map any unknown routes. Parameters passed to `mapUnknownRoutes()` can be:

* A string to a moduleId. This module will be navigated to any time a route is not found.
* A routeConfig object. This configuration object will be used any time a route is not found.
* A function which is passed the NavigationInstruction object and can decide the route dynamically.

### Using a ModuleId for Unknown Routes

<code-listing heading="Static Unknown Routes">
  <source-code>
  <source-code lang="ES 2015/2016">

    export class App {
      configureRouter(config, router) {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true, caseSensitive: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' }
        ]);

        config.mapUnknownRoutes('not-found');
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index', nav: true, caseSensitive: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' }
        ]);

        config.mapUnknownRoutes('not-found');
      }
    }
  </source-code>
</code-listing>

The above example will redirect any unmatched routes to the `not-found` component module.

### Using A Function For Unknown Routes

The function passed to `mapUnknownRoutes()` has to return:

* A string to a moduleId.
* An object with property `moduleId` of type string.
* A `RouteConfig` object.
* A `Promise` that resolves to any of the above.

<code-listing heading="Dynamic Unknown Routes">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        this.router = router;

        config.title = 'Aurelia';

        var handleUnknownRoutes = (instruction) => {
            return { route: 'not-found', moduleId: 'not-found' };
        }

        config.mapUnknownRoutes(handleUnknownRoutes);

        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }

  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, NavigationInstruction, Router, RouteConfig} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        this.router = router;

        config.title = 'Aurelia';

        let handleUnknownRoutes = (instruction: NavigationInstruction): RouteConfig => {
            return { route: 'not-found', moduleId: 'not-found' };
        }

        config.mapUnknownRoutes(handleUnknownRoutes);

        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }
  </source-code>
</code-listing>

## [Redirecting Routes](aurelia-doc://section/7/version/1.0.0)

Aurelia allows redirecting of routes to URL fragments by specifying redirect with a string consisting of a URL fragment.

<code-listing heading="Route Config Redirects">
  <source-code lang="ES 2015/ES 2016/TypeScript">
    config.map([
      { route: '',           redirect: 'home' },
      { route: 'home',       name: 'home',       moduleId: 'home/index' }
    ]);
  </source-code>
</code-listing>

> Info: Use Redirect On Empty Routes with Child Routers
> The `redirect` is particularly useful when you have an "empty" route pattern (such as the first route above) that maps to a component with a child router. In this case, create a non-empty route and then redirect the empty route to the non-empty route (as above). This will enable the child router to consistently match child routes without getting confused in scenarios where the empty route was matched.

## [Pipelines](aurelia-doc://section/8/version/1.0.0)

Aurelia has two router classes, `AppRouter` and `Router`. `AppRouter` extends the `Router` class and is the main application router. `Router` is used for any child routers including nested child routers. One of the main differences between the two is pipelines are only allowed on the `AppRouter` and not any child routers.

You can create your own pipeline steps using `addPipelineStep`, but the step's name must match one of the pipeline's slots, the default slots in order are `authorize`, `preActivate`, `preRender`, and `postRender`. Aurelia also has functions for creating a pipeline step for these slots.

* `authorize` is called between loading the route's step and calling the route view-model' `canActivate` function if defined.
* `preActivate` is called between the route view-model' `canActivate` function and the previous route view-model's `deactivate` function if defined.
* `preRender` is called between the route view-model's activate function and before the component is rendered/composed.
* `postRender` is called after the component has been render/composed.

A pipeline step must be an object that contains a `run(navigationInstruction, next)` function.

<code-listing heading="An Authorize Step">
  <source-code lang="ES 2015/2016">
    import {Redirect} from 'aurelia-router';

    export class App {
      configureRouter(config, router) {
        var step = new AuthorizeStep;
        config.addAuthorizeStep(step)
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index', settings: { auth: true } },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail', settings: { auth: true } },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files', nav: true }
        ]);
      }
    }

    class AuthorizeStep {
      run(navigationInstruction, next) {
        if (navigationInstruction.getAllInstructions().some(i => i.config.settings.auth)) {
          var isLoggedIn = // insert magic here;
          if (!isLoggedIn) {
            return next.cancel(new Redirect('login'));
          }
        }

        return next();
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {Redirect, NavigationInstruction, RouterConfiguration, Router, Next} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        config.title = 'Aurelia';
        config.addPipelineStep('authorize', AuthorizeStep);
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }

    class AuthorizeStep {
      run(navigationInstruction: NavigationInstruction, next: Next): Promise<any> {
        if (navigationInstruction.getAllInstructions().some(i => i.config.settings.auth)) {
          var isLoggedIn = //insert magic here;
          if (!isLoggedIn) {
            return next.cancel(new Redirect('login'));
          }
        }

        return next();
      }
    }
  </source-code>
</code-listing>

<code-listing heading="A PreActivate Step">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        function step() {
          return step.run;
        }
        step.run = (navigationInstruction, next) => {
          return next()
        };
        config.addPreActivateStep(step)
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {Redirect, NavigationInstruction, RouterConfiguration} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration): void {
        function step() {
          return step.run
        }
        step.run = (navigationInstruction: NavigationInstruction, next: Function): Promise<any> {
          return next();
        };

        config.title = 'Aurelia';
        config.addPreActivateStep(step);
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }
  </source-code>
</code-listing>

<code-listing heading="A PreRender Step">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        var step = {
          run: (navigationInstruction, next) => {
            return next()
          }
        };
        config.addPreRenderStep(step);
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {Redirect, NavigationInstruction, RouterConfiguration} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration): void {
        let step = {
          run: (navigationInstruction: NavigationInstruction, next: Function): Promise<any> {
            return next();
          }
        };

        config.title = 'Aurelia';
        config.addPreRenderStep(step);
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }
  </source-code>
</code-listing>

<code-listing heading="A PostRender Step">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        var step = {
          run: (navigationInstruction, next) => {
            return next();
          }
        };
        config.addPostRenderStep(step);
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {Redirect, NavigationInstruction, RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {

        let step = {
          run: (navigationInstruction: NavigationInstruction, next: Function): Promise<any> {
            return next();
          }
        };

        config.title = 'Aurelia';
        config.addPostRenderStep(step);
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
          { route: 'files/*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
        ]);
      }
    }
  </source-code>
</code-listing>

## [Rendering View Ports](aurelia-doc://section/9/version/1.0.0)

Every instance of a `router-view` custom element essentially defines a "view port".  When you give a `router-view` a name, you can refer to it in the `viewPorts` property of the route configuration in your javascript.  The value of a `viewPorts` property is an object where each property name is the name of a view port (ie, `router-view`) and each value is the `moduleId` destination of the route.  Thus you can specify any number of view ports on a single route configuration.

> Info
> If you don't name a `router-view`, it will be available under the name 'default'.

Following is an example of the use of view ports:

<code-listing heading="app.html">
  <source-code lang="HTML">
    <template>
      <div>
        <router-view name="left"></router-view>
      </div>
      <div>
        <router-view name="right"></router-view>
      </div>
    </template>
  </source-code>
</code-listing>

<code-listing heading="app${context.language.fileExtension}">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        config.title = 'Aurelia';
        config.map([
          { route: 'users', name: 'users', viewPorts: { left: { moduleId: 'user/list' }, right: { moduleId: 'user/detail' } } }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {Redirect, NavigationInstruction, RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        config.title = 'Aurelia';
        config.map([
          { route: 'users', name: 'users', viewPorts: { left: { moduleId: 'user/list' }, right: { moduleId: 'user/detail' } } }
        ]);
      }
    }
  </source-code>
</code-listing>

> Info
> In addition to the `moduleId`, you can also specify a "layout" in the configuration of a view port.  See the discussion of Layouts elsewhere in this document.

## [Layouts](aurelia-doc://section/10/version/1.0.0)

Similar to MVC-style master/layout pages, Aurelia allows you to use a "layout" view like an MVC "master template" for a set of views. 

The set of views subject to being part of a layout is defined in Aurelia as a set of views referenced by one or more routes in a router configuration.  There are two ways to associate a layout with routes.  The first is via HTML, the second is via view model code.

> Info
> We're going to be a little sloppy here in terminology.  Technically, routes refer to "moduleIds", not 
"views".  Since the router resolves a moduleId to a view, indirectly the router does reference a view.  It is easy to picture a view visually contained within a layout, so in this topic to we'll refer to views referenced by a route, not modules.

We'll look at using HTML first.  We know that the `router-view` custom HTML element is always associated with a set of one or more views referenced in a router configuration given in its parent view's view model.  By associating a layout with a `router-view` one can thus associate a layout with the same set of views with which the `router-view` is associated.  

To specify a layout on the `router-view` custom element, we use the following attributes:

* `layout-view` - specifies the file name (with path) of the layout view to use.
* `layout-view-model` - specifies the moduleId of the view model to use with the layout view.
* `layout-model` - specifies the model parameter to pass to the layout view model's `activate` function.

> Info
> All of these layout attributes are bindable.

Following is an example of HTML in which we specify that we want all destination views reachable under the `router-view` to be laid-out inside a view with file name `layout.html`, located in the same directory as the view contianing the `router-view`:

<code-listing heading="app.html">
  <source-code lang="HTML">
    <template>
      <div>
        <router-view layout-view="layout.html"></router-view>
      </div>
    </template>
  </source-code>
</code-listing>

Here is the layout view itself:

<code-listing heading="layout.html">
  <source-code lang="HTML">
    <template>
      <div class="left-content">
        <slot name="left-content"></slot>
      </div>
      <div class="right-content">
        <slot name="right-content"></slot>
      </div>
    </template>
  </source-code>
</code-listing>

And here we define a view that we want to appear within the layout:

<code-listing heading="home.html">
  <source-code lang="HTML">
    <template>
      <div slot="left-content">
        <p>${leftMessage}.</p>
      </div>
      <div slot="right-content">
        <p>${rightMessage}.</p>
      </div>
      <div>This will not be displayed in the layout because it is not contained in any named slot referenced by the layout.</div>
    </template>
  </source-code>
</code-listing>

<code-listing heading="home${context.language.fileExtension}">
  <source-code lang="ES 2015/2016">
    export class Home {
      constructor() {
        this.leftMessage = "I'm content that will show up on the left";
        this.rightMessage = "I'm content that will show up on the right";
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    export class Home {
      constructor() {
        this.leftMessage = "I'm content that will show up on the left";
        this.rightMessage = "I'm content that will show up on the right";
      }
    }
  </source-code>
</code-listing>

Observe how we use the `slot` mechanism for associating parts of the layout to parts of the views that are to be contained within the layout.  (Happy for developers, this is conveniently the same mechanism and syntax we use in Aurelia when providing content to custom elements.)

Now we just have to define the route configuration that will be associated with the `router-view`:

<code-listing heading="app${context.language.fileExtension}">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router){
        config.map([
          { route: '', name: 'home', moduleId: 'home' }
        ]);

        this.router = router;
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {){
        config.map([
          { route: '', name: 'home', moduleId: 'home' }
        ]);

        this.router = router;
      }
    }
  </source-code>
</code-listing>

Thus when we navigate to the module "home" we find that it is laid-out as desired inside the layout view.

Note there is nothing different about the above route configuration with or without the layout.  It may reference any number of views that would all be included by default in the layout. 

So that is how we use HTML to associate a layout view with a set of views referenced in a router configuration.  

We can also associate layouts with route configurations using code in our view model.  Suppose we like what we've done above, but we have a couple views that we would like to associate with a different layout and would thus like to partially override the configuration given in the HTML.  The following code is an example of how we can do that:

<code-listing heading="app${context.language.fileExtension}">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router){
        config.map([
          { route: '', name: 'home', moduleId: 'home' },
          { route: 'login', name: 'login', moduleId: 'login/index', layoutView: 'layout-login.html' },
          { route: 'users', name: 'users', moduleId: 'users/index', layoutModel: 'layout-users', layoutViewModel: { access: "admin" } }
        ]);

        this.router = router;
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {){
        config.map([
          { route: '', name: 'home', moduleId: 'home' },
          { route: 'login', name: 'login', moduleId: 'login/index', layoutView: 'layout-login.html' },
          { route: 'users', name: 'users', moduleId: 'users/index', layoutModel: 'layout-users', layoutViewModel: { access: "admin" } }
        ]);

        this.router = router;
      }
    }
  </source-code>
</code-listing>

The above example will assign different layouts to the "login" and "users" views, overriding the HTML while leaving "home" to remain as configured in the HTML.  Noticing we're using camel-cased property names here, unlike in the HTML.  

You can also specify a layout in the `viewPorts` configuration of a route.  See a simple example, below:

<code-listing heading="app.html">
  <source-code lang="HTML">
    <template>
      <div>
        <router-view name="myRouterView"></router-view>
      </div>
    </template>
  </source-code>
</code-listing>

<code-listing heading="app${context.language.fileExtension}">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router){
        config.map([
          { route: '', name: 'home', viewPorts: { myRouterView: { moduleId: 'home', layoutView: 'default.html' } } }
        ]);

        this.router = router;
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {){
        config.map([
          { route: '', name: 'home', viewPorts: { myRouterView: { moduleId: 'home', layoutView: 'default.html' } } }
        ]);

        this.router = router;
      }
    }
  </source-code>
</code-listing>

## [View Swapping and Animation](aurelia-doc://section/11/version/1.0.0)

When the Aurelia router navigates from one view to another, we refer to this as "swapping" one view for another.  Aurelia gives us an optional set of strategies dictating how a swap proceeds, or more specifically, how animation plays out during the swap.  We refer to these strategies more precisely as the "swap order".

> Info
> If there is no animation defined, then swap-order has no visible impact.

You can apply a swap strategy to one or more routes by applying the `swap-order` attribute to a `router-view` custom HTML element.  The strategy will then be applied in any transition between two views accessible under the `router-view`.  

> Info
> `swap-order` is bindable.

The following swap order strategies are available:

* before - animate the next view in before removing the current view
* with - animate the next view at the same time the current view is removed
* after - animate the next view in after the current view has been removed (the default)

Here is an example of setting the swap order strategy on a `router-view`:
 
<code-listing heading="swap-order">
  <source-code lang="HTML">
    <template>
      <div>
        <router-view swap-order="before"></router-view>
      </div>
    </template>
  </source-code>
</code-listing>


## [Internationalizing Titles](aurelia-doc://section/12/version/1.0.0)

If your application targets multiple cultures or languages, you probably want to translate your route titles. The `Router` class has a `transformTitle` property that can be used for this. It is expected to be assigned a function that takes the active route's title as a parameter and then returns the translated title. For example, if your app uses `aurelia-i18n`, its routes' titles would typically be set to some translation keys
and the `AppRouter`'s `transformTitle` would be configured in such a way that the active route's title is translated using the `I18N`'s `tr` method. Additionally you can listen to a custom event published by the I18N service to react on locale changes using the EventAggregator:

<code-listing heading="src/main${context.language.fileExtension}">
  <source-code lang="ES 2015/2016">
    import Backend from 'i18next-xhr-backend';
    import {AppRouter} from 'aurelia-router';
    import {EventAggregator} from 'aurelia-event-aggregator';

    export function configure(aurelia) {
      aurelia.use
        .standardConfiguration()
        .plugin('aurelia-i18n', i18n => {
          i18n.i18next.use(Backend);

          return i18n.setup({
            backend: {
              loadPath: './locales/{{lng}}.json',
            },
            lng : 'en',
            fallbackLng : 'en'
          }).then(() => {
            const router = aurelia.container.get(AppRouter);
            router.transformTitle = title => i18n.tr(title);
            
            const eventAggregator = aurelia.container.get(EventAggregator);
            eventAggregator.subscribe('i18n:locale:changed', () => {
              router.updateTitle();
            });
          });
        });

      aurelia.start().then(() => aurelia.setRoot());
    }
  </source-code>
  <source-code lang="TypeScript">
    import Backend from 'i18next-xhr-backend';
    import {AppRouter} from 'aurelia-router';
    import {EventAggregator} from 'aurelia-event-aggregator';

    export function configure(aurelia) {
      aurelia.use
        .standardConfiguration()
        .plugin('aurelia-i18n', i18n => {
          i18n.i18next.use(Backend);

          return i18n.setup({
            backend: {
              loadPath: './locales/{{lng}}.json',
            },
            lng : 'en',
            fallbackLng : 'en'
          }).then(() => {
            const router = aurelia.container.get(AppRouter);
            router.transformTitle = title => i18n.tr(title);
            
            const eventAggregator = aurelia.container.get(EventAggregator);
            eventAggregator.subscribe('i18n:locale:changed', () => {
              router.updateTitle();
            });
          });
        });

      aurelia.start().then(() => aurelia.setRoot());
    }
  </source-code>
</code-listing>
<code-listing heading="locales/en.json">
  <source-code lang="JSON">
    {
      "titles": {
        "app": "My App",
        "home": "Home"
      }
    }
  </source-code>
</code-listing>
<code-listing heading="src/app${context.language.fileExtension}">
  <source-code lang="ES 2015/2016">
    export class App {
      configureRouter(config, router) {
        this.router = router;
        config.title = 'titles.app';
        config.map([
          { route: ['', 'home'], name: 'home', moduleId: 'home', title: 'titles.home' }
        ]);
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        this.router = router;
        config.title = 'titles.app';
        config.map([
          { route: ['', 'home'], name: 'home', moduleId: 'home', title: 'titles.home' }
        ]);
      }
    }
  </source-code>
</code-listing>

The default value of the`transformTitle` property does the following:

* For the child `Router`, it delegates to its parent's `transformTitle` function.
* For the `AppRouter`, it returns the title untransformed.

In the previous example, the `AppRouter`'s `transformTitle` is set, so all child `Router`s will delegate down to it by default. However, this means that the `transformTitle` can be overridden for specific child `Router`s if some areas of your app need custom transformation.

## [Configuring a Fallback Route](aurelia-doc://section/12/version/1.0.0)

Whenever navigation is rejected, it is redirected to a previous location. However in certain cases a previous location doesn't exist, e.g. when it happens as the first navigation after the startup of application. To handle this scenario, you can set up a fallback route.

<code-listing heading="app${context.language.fileExtension}">
  <source-code lang="ES 2015/2016">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config, router) {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true, caseSensitive: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' }
        ]);

        config.fallbackRoute('users');
      }
    }
  </source-code>
  <source-code lang="TypeScript">
    import {RouterConfiguration, Router} from 'aurelia-router';

    export class App {
      configureRouter(config: RouterConfiguration, router: Router): void {
        this.router = router;
        config.title = 'Aurelia';
        config.map([
          { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
          { route: 'users',            name: 'users',      moduleId: 'users/index', nav: true, caseSensitive: true },
          { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' }
        ]);

        config.fallbackRoute('users');
      }
    }
  </source-code>
</code-listing>
