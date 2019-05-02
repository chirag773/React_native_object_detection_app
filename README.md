**Object detection React-Native App**

1. Setup and Installation 
2. working

**1. Setup**

* `git clone https://github.com/chirag773/React_native_object_detection_app.git`

* Change directory

* `cd React_native_object_detection_app`

* If you are using `yarn` then just run the following command to install all dependecies `yarn`

* If you are using `npm` the run the followung command to install all the dependencies `npm install`

**2. working**

We will start from importing all the necessary dependencies

importing camera for using camera
importing permission to access camera 


```js
import React from 'react';
import { Text, View, TouchableOpacity, FlatList } from 'react-native';
import { Camera, Permissions, ImageManipulator } from 'expo';

```

Now the main and important part We are going to use (Clarifai)[https://clarifai.com/] API ehich have pre-trained model of many different classes.

Go to the website and sign In you will get API-KEY copy and paste it in the place of api key in the code.

ONce you have done with API-KEY you can now use it. Remember to confirm it from Gmail otherwise you will only get 100 request only.  

```js
const Clarifai = require('clarifai');

const clarifai = new Clarifai.App({
  apiKey: 'Your API key',
});
process.nextTick = setImmediate;

```

Take a deep look to the below code.

We will start by creating our state.

hasCameraPermission will manage if the permission grantent from the user
we will create empty array of predictions where our all the prediction will gonna store.

```js
state = {
    hasCameraPermission: null,
    predictions: [],
  };
```

This componentWillMount life cycle method check the status if granted or not .

```js
 async componentWillMount() {
    const { status } = await Permissions.askAsync(Permissions.CAMERA);
    this.setState({ hasCameraPermission: status === 'granted' });
  }
```

The below code will Check if the user had given the permisson for accesing the camera.
if not it will return false.

```js
 const { hasCameraPermission, predictions } = this.state;
    if (hasCameraPermission === null) {
      return <View />;
    } else if (hasCameraPermission === false) {
      return <Text>No access to camera</Text>;
    } 
```

and if yes it will return below code


```js
return (
        <View style={{ flex: 1 }}>
          <Camera
            ref={ref => {
              this.camera = ref;
            }}
            style={{ flex: 1 }}
            type={this.state.type}
          >
            <View
              style={{
                flex: 1,
                backgroundColor: 'transparent',
                flexDirection: 'column',
                justifyContent: 'flex-end'
              }}
            >
              <View
                style={{
                  flex: 1,
                  alignSelf: 'flex-start',
                  alignItems: 'center',
                }}
              >
                <FlatList
                  data={predictions.map(prediction => ({
                    key: `${prediction.name} ${prediction.value}`,
                  }))}
                  renderItem={({ item }) => (
                    <Text style={{ paddingLeft: 15, color: 'white', fontSize: 20 }}>{item.key}</Text>
                  )}
                />
              </View>
              <TouchableOpacity
                style={{
                  flex: 0.1,
                  alignItems: 'center',
                  backgroundColor: 'blue',
                  height: '10%',
                }}
                onPress={this.objectDetection}
              >
                <Text style={{ fontSize: 30, color: 'white', padding: 15 }}>
                  {' '}
                  Detect Objects{' '}
                </Text>
              </TouchableOpacity>
            </View>
          </Camera>
        </View>
      );
```

In the above code while `onPress={this.objectDetection}` function will run that function will have child function.

In the below code `this.capturePhoto()` will capture the photo when button is press and stored it into a photo variable.

`this.resize(photo)` take the capture phot as a argument it will resize the photo and save it into resized variable.

`this.predict(resized)` it will predict the result of current captured photo and save it into predictions variable and then will set state `this.setState({ predictions: predictions.outputs[0].data.concepts })`

```js
objectDetection = async () => {
    let photo = await this.capturePhoto();
    let resized = await this.resize(photo);
    let predictions = await this.predict(resized);
    this.setState({ predictions: predictions.outputs[0].data.concepts });
  };
```
We are using await and async 

```js
capturePhoto = async () => {
    if (this.camera) {
      let photo = await this.camera.takePictureAsync();
      return photo.uri;
    }
  };
  resize = async photo => {
    let manipulatedImage = await ImageManipulator.manipulateAsync(
      photo,
      [{ resize: { height: 300, width: 300 } }],
      { base64: true }
    );
    return manipulatedImage.base64;
  };
  predict = async image => {
    let predictions = await clarifai.models.predict(
      Clarifai.GENERAL_MODEL,
      image
    );
    return predictions;
  };
```




