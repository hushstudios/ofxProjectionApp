# ofxProjectionApp Overview

ofxProjectionApp is a basic template for a projection mapping system using a variety of different addons to simplify getting a projection mapping system up quickly. 

The intended process is to draw your app into an FBO that acts as your canvas, and once you are ready to projection, use this setup. Note that the application does not do any scaling; therefore, if you tell the application to draw three warps at 4k and run your application at 

## Feature List / Addons Needed

* [ofxWarp](https://github.com/local-projects/ofxWarp): create editable linear, bi-linear, and perspective warps. Note this is using the local-projects fork of [prisonerjohn's addon](https://github.com/prisonerjohn/ofxWarp) and the [feature/multipleControls](https://github.com/local-projects/ofxWarp/tree/feature/multipleControls) branch. The feature/multipleControls branch is working for bi-linear warps, but the new features still need to be cascaded to to the linear and perspective warp classes. New features include: clicking control points to toggle active state instead of hovering, controling multiple control points at once
* [ofxDatGui](https://github.com/local-projects/ofxDatGui): In application GUI 
* [ofxJSON](https://github.com/local-projects/ofxJSON): configurable json files
* [ofxNotificationCenter](https://github.com/local-projects/ofxNotificationCenter): addon to send and receive messages to a an application wide
* Saves and loads projection settings dynamically

##Compatibility
* openFrameworks 0.9 and up
* OpenGL 3 and up (programmable pipeline)
* The included shaders only work with normalized textures (GL_TEXTURE_2D) but can be easily modified to work with rectangle textures

## Class Overview
ofxProjectionApp is more of a template then a singular addon with one function; this results in a lot of moving parts. Below is a WIP diagram of the overall class structure.



## Quick Setup Guide 
	
1. Set open gl version in `int(){}`. This needs to be OpenGL 3 and up.

	```
	winSettings.setGLVersion(3, 2);
	```
2. Call `ofDisableArbTex()` in `ofApp::setup()`
3. Create an `ofxFbo` object to draw in. This will get passed to the projection manager.

	```
	\\ In Header
	ofFbo *canvas;
	
	 ofFbo::Settings fboSettings;
    fboSettings.width = ofGetWidth();
    fboSettings.height = ofGetHeight();
    fboSettings.useDepth = false;
    fboSettings.textureTarget = GL_TEXTURE_RECTANGLE_ARB; //mipmaps!
    fboSettings.numSamples = 8;
    fboSettings.numColorbuffers = 1;
    
    canvas = new ofFbo();
    canvas->allocate(fboSettings);
    canvas->begin();
    ofClear(0,0,0,0);
    canvas->end();
	``` 
4. Create your scene using `ofxInterface`.

	```
	 ofxInterface::Node* scene;
	  scene = new ofxInterface::Node();
    scene->setSize(ofGetWidth(), ofGetHeight());
    scene->setPosition(0, 0);
    scene->setName("scene");
    scene->setEnabled(true);
	``` 
	
5. Create an `ofxProjectionApp` object which is essentialy going to be the projection manager.

	```
	// In Header
	ofxProjectionApp * projections;
	
	// In cpp file
	projections = new ofxProjectionApp();
	```

6.  Set up the projector specs.

	```
	// In cpp file
	ProjectorManager::one().addProjector(0, //order
                                         2, //numWarps
                                         ofVec2f(ofGetWidth(), ofGetHeight()), //projector resolution
                                         ofVec2f(0.0f, 0.0f)); // projector position
	```
7. Finish the setup for `ofxProjectionApp`.

	```
    projections->setup(canvas, //ofFbo * _canvasRef
                       false, //bool _loadFromFile
                       ofVec2f(ofGetWidth(), ofGetHeight()), //ofVec2f _appSize
                       1.0f, //float _scaleDenominator
                       scene, //ofxInterface::Node* _sceneRef
                       ""); //string _directoryPath
	```

8. Add any applicaiton states and set up the GUI manager with a vector of those states in string format. State management is flexible, so it's really up to you on how you want to use it. See advanced example for templated version.
	
	```
   vector<string> appStates;
    
    for(int i = 0; i < AppStates::NUM_STATES; i++)
    {
        string temp = getStateName(static_cast<AppStates>(i));
        appStates.push_back(temp);
    }
    
    projections->setupGuiManager(appStates);
    ```
9.  Set up warps for `ofxProjectionApp`
	
	```
	projections->setupWarps();
	```
10. Set up the `TouchManager` for `ofxInterface`
	```
	ofxInterface::TouchManager::one().setup();
	```
11. Update the scene, `ofxProjectionApp` object, and `TouchManager` in `ofApp::update()`
	
	```
	float dt = 1./60.;
    
    //Update Scene.
    ofxInterface::TouchManager::one().update();
    
    //projections->update();
    scene->updateSubtree(1./60.);
    
    projections->update();
	```	 
13.  Draw your content in canvas
	
	```
	 /*
     Draw canvas to the fbo
     */
    canvas->begin();
    {
        //Draw whatever you want here!
        ofClear(0,0,0,0);
        
        ofSetColor(0.0f);
        ofDrawRectangle(0.0f, 0.0f, canvas->getWidth(), canvas->getWidth()); 
        
        ofSetColor(ofColor::pink);
        float size = 100.0f;
        ofDrawCircle(ofGetWidth()/2 - size/2, ofGetHeight()/2 - size/2, size, size);
        
    }
    canvas->end();
    ```

## Key Controls

* `w`: toggles warp editing mode
* `alt` + `left-click`: selects warp
* `left-click` on selected warp: selects closest control point, note that you can click on multiple control points at once ad shift them
* `shift` + `left-click`: deselects control point
* `right-click` on warp: brings up edge blend gui

Once you are in warp editing mode: 

For Bilinear warps only:

* m to toggle between linear and curved mapping
* F1 to reduce the number of horizontal control points
* F2 to increase the number of horizontal control points
* F3 to reduce the number of vertical control points
* F4 to increase the number of vertical control points
* F5 to decrease the mesh resolution
* F6 to increase the mesh resolution

## Class Overview

