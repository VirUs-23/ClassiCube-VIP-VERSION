Visual studio unsupported platform toolset
---------------------
To fix the ```Error MSB8036 The Windows SDK version 5.1 was not found. Install the required version of Windows SDK or change the SDK version in the project property pages or by right-clicking the solution and selecting "Retarget solution"```
* Right click **ClassiCube** project (it's under the *Solution 'ClassiCube'* in the *Solution Explorer* pane)
* Click **Properties**
* Make sure you are in **General** tab under **Configuration Properties**
* You should see a dropdown named **Platform Toolset**. Click on it.
* Change it to one of the toolsets in the list that pops up.
* Click **OK**. You should be able to compile now

![image](https://user-images.githubusercontent.com/6509348/60266950-727e4780-992c-11e9-98fb-85eb34959e93.png)

Common compilation errors
---------------------
#### Undefined reference to 'clock_gettime'
Add ```-lrt``` when compiling. Occurs when using glibc versions before 2.17.

#### fatal error: execinfo.h: No such file or directory
Install ```libexecinfo``` package. Occurs when using musl.

#### Undefined reference to 'backtrace'
Add ```-lexecinfo``` when compiling. Occurs when using musl.

Webclient patches
---------------------
#### Starting game **Error 00000002 when setting current directory**
This is caused by IndexedDB not being initialised, which also means saved maps are lost when the tab is closed.

Due to how IndexedDB works, you must load and initialise it before the game starts. Change:

```
    <script type='text/javascript'>
      var Module = {
        preRun: [],
...
```
to
```
    <script type='text/javascript'>
      // need to load IndexedDB before running the game
      function preloadIndexedDB() {
        addRunDependency('load-idb');
        FS.mkdir('/classicube');
        FS.mount(IDBFS, {}, '/classicube');
        FS.syncfs(true, function(err) { 
            if (err) window.cc_idbErr = err; 
            removeRunDependency('load-idb');
        })
      }

      var Module = {
        preRun: [ preloadIndexedDB ],
...
```

#### Mouse scrolling not properly prevented
With recent chrome/firefox versions, page is still scrolled and console is spammed with\
```"[Intervention] Unable to preventDefault inside passive event listener due to target being treated as passive."```

You need to to register events as a passive handler. Look for something like:
```
eventHandler.target.addEventListener(eventHandler.eventTypeString, jsEventHandler, eventHandler.useCapture);
```
and change to 
```
eventHandler.target.addEventListener(eventHandler.eventTypeString, jsEventHandler, { useCapture: eventHandler.useCapture, passive: false });
```

#### Texture pack confirm dialog always shows *Download size: Determining*..
Unfortunately emscripten doesn't store content-length for HEAD http requests. This can be fixed.

First you need to find at what offset emscripten stores content-length. Look for something like:
```
xhr.onprogress = function(e) {
...
      Fetch.setu64(fetch + 32, e.total);
```
then look for
```
xhr.onload = function(e) {
...
```
and finally change that to
```
xhr.onload = function(e) { 
    Fetch.setu64(fetch + 32, e.total);
...
```
