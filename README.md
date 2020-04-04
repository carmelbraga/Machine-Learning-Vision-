# Machine-Learning-Vision-

The detectScene function takes an "image" parameter of type CIImage and then detects the contents of said image by using a machine learning model especifically made for this, there is also a "displayString" method to let the user know the action is being performed: 

 ```swift
   func detectScene(image: CIImage) {
        displayString(string: "detecting scene...")
  ```

The first step is to load the chosen machine learning model by assigning it to the constant "model" through the Swift VNCoreMLModel class, there is also a "displayString" method that is user in case of an error:
        
       
 ```swift
   guard let model = try? VNCoreMLModel(for: VGG16().model) else {
            displayString(string: "Can't load ML model.")
            return
        }
  ```

 
The second step is to create an imagine analysis (Vision) request by assigning it to the constant "request" through the Swift class VNCoreMLRequest that takes the previously assigned "model" constant that contains the VGG16() model that will be used for this. To assist with the request, there is a completion handler to specify a method to receive results from the model after you run the request as well as a "displayString" method to alert the user if an error occurs. Lastly, if no errors occur, the “results” property will cointain VNClassificationObservation objects identified by the model:
        
        
 ```swift
    let request = VNCoreMLRequest(model: model) { [weak self] request, error in
            guard let results = request.results as? [VNClassificationObservation],
                let _ = results.first else {
                    self?.displayString(string: "Unexpected result type from VNCoreMLRequest")
                    return
            }
  ```
  
The third step or the second part of the second step is to update the label with our new-found results. We use DispatchQueue.main.async to acess the main thread as it's required to update any UI elements as well as stop the activity indicator (activityIndicator) since the completion handler will have started it by the time we get to running the code. There is also a "displayString" method that will iterate through each result and alert the user if a error is encountered:

```swift
  DispatchQueue.main.async { [weak self] in
                self?.activityIndicator.stopAnimating()
                for result in results {
                    self?.displayString(string: "\(Int(result.confidence * 100))% \(result.identifier)")
                }
            }
        }
   ```
 
The fourth and last step is to actually perform the request by starting the acitvity indicator (activityIndicator) and creating the handler. The image is then passed to the handler and the do-catch block will then attempt the array of requests ([request]) by using the "perform" property while handling any errors that might be thrown. If any errors occur, the activity indicator (activityIndicator) will be stopped and the UI will be updated. If successful, the request completion handler is called and detectScene is completed:
       
       
 ```swift
   activityIndicator.startAnimating()
        
        let handler = VNImageRequestHandler(ciImage: image)
        DispatchQueue.global(qos: .userInteractive).async {
            do {
                try handler.perform([request])
            } catch {
                DispatchQueue.main.async { [weak self] in
                    self?.displayString(string: error.localizedDescription)
                    self?.activityIndicator.stopAnimating()
                }
            }
        }
    }

}
  ```
