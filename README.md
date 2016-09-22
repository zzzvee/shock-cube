# shock-cube
ArduinoInput input; // Import our ArduinoInput Class 
SoundInput sound;

// Create shader objects
PShader shaderToy;
PShader pixelateShader;
PShader blurShader;
PShader rippleShader;
PShader rgbShiftShader;

// Create off sceen textures to render our shaders into
PGraphics shaderToyFBO;
PGraphics pixelateFBO;
PGraphics blurFBO;
PGraphics rippleFBO;
PGraphics rgbShiftFBO;

//-------------------------------------
void setup() {
  size(640, 480, P3D);
  noStroke();
  background(0);
  
  input = new ArduinoInput(this); // call the constructor of ArduinoInput
  sound = new SoundInput(this);
 
  shaderToy = loadShader("zShader.glsl"); // Load our .glsl shader from the /data folder
  shaderToy.set("iResolution", float(width), float(height), 0); // Pass in our xy resolution to iResolution uniform variable in our shader
  shaderToyFBO = createGraphics(width, height, P3D);
  shaderToyFBO.shader(shaderToy);
  shaderToy.set("speed", 3);
  shaderToy.set("cirum", 1.5);
  shaderToy.set("wobble", 4.);

  pixelateShader = loadShader("pixelate.glsl");
  pixelateShader.set("iResolution", float(width), float(height), 0); 
  pixelateFBO = createGraphics(width, height, P3D);
  pixelateFBO.shader(pixelateShader);

  blurShader = loadShader("blur.glsl");
  blurShader.set("iResolution", float(width), float(height), 0); 
  blurFBO = createGraphics(width, height, P3D);
  blurFBO.shader(blurShader);

  rippleShader = loadShader("ripple.glsl");
  rippleShader.set("iResolution", float(width), float(height), 0); 
  rippleFBO = createGraphics(width, height, P3D);
  rippleFBO.shader(rippleShader);

  rgbShiftShader = loadShader("chromaticAbberation.glsl");
  rgbShiftShader.set("iResolution", float(width), float(height), 0);
  rgbShiftFBO = createGraphics(width, height, P3D);
  rgbShiftFBO.shader(rgbShiftShader); 
}

//-------------------------------------
void updateShaderParams() {
  float[] sensorValues = input.getSensor(); 
  
  shaderToy.set("dryWet", 0.0);
  blurShader.set("blurAmount", 0.0);
  pixelateShader.set("downsample", 0.00);
  rippleShader.set("frequency", map(sensorValues[0],0.0,1024.0,0.0,1.0));
  rippleShader.set("waveNum", map(sensorValues[1],0.0,1024.0,0.0,1.0));
  rgbShiftShader.set("offset",map(sound.getVolume(),0.0,1.0,0.0,1.0));
  shaderToy.set("speed", map(sensorValues[2],0.0,1024.0,3.0,0.005));
  shaderToy.set("cirum", map(sensorValues[0],0.0,1024.0,1.5,7.5));
  shaderToy.set("wobble", map(sensorValues[1],0.0,1024.0,4.0,0.8));
}

//-------------------------------------
void draw() {
  updateShaderParams();

  shaderToyFBO.beginDraw();
  shaderToy.set("iGlobalTime", millis() / 1000.0); // pass in a millisecond clock to enable animation 
  shader(shaderToy); 
  shaderToyFBO.rect(0, 0, width, height); // We draw a rect here for our shader to draw onto
  shaderToyFBO.endDraw();

  pixelateFBO.beginDraw();
  pixelateShader.set("iGlobalTime", millis() / 1000.0);
  pixelateShader.set("tex", shaderToyFBO);
  shader(pixelateShader); 
  pixelateFBO.rect(0, 0, width, height); 
  pixelateFBO.endDraw();

  blurFBO.beginDraw();
  blurShader.set("iGlobalTime", millis() / 1000.0); 
  blurShader.set("tex", pixelateFBO);
  shader(blurShader); 
  blurFBO.rect(0, 0, width, height); 
  blurFBO.endDraw();

  rippleFBO.beginDraw();
  rippleShader.set("iGlobalTime", millis() / 1000.0); 
  rippleShader.set("tex", blurFBO);
  shader(rippleShader); 
  rippleFBO.rect(0, 0, width, height); 
  rippleFBO.endDraw();

  rgbShiftFBO.beginDraw();
  rgbShiftShader.set("iGlobalTime", millis() / 1000.0); 
  rgbShiftShader.set("tex", rippleFBO);
  shader(rgbShiftShader); 
  rgbShiftFBO.rect(0, 0, width, height); 
  rgbShiftFBO.endDraw();

  image(rgbShiftFBO, 0, 0, width, height);
}
