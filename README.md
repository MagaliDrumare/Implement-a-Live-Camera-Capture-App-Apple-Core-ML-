
 ![alt tag](https://github.com/MagaliDrumare/Implement-a-live-camera-capture-App-using-Apple-Core-ML-/blob/master/gifstreamlive.gif)

# Implement a live camera capture App using Apple Core ML
* Mastering Core ML for iOS on Udemy by Mohammad Azam : http://bit.ly/2ADSykK

``` swift
//  ViewController.swift

//  Created by DRUMARE
//  Credits © 2017 Mohammad Azam


import UIKit
import AVFoundation
import CoreML
import Vision

class ViewController: UIViewController, AVCaptureVideoDataOutputSampleBufferDelegate {
    
    // Create the image and text containers
    @IBOutlet weak var imageView :UIImageView!
    @IBOutlet weak var textView :UITextView!
    
    //Import the inceptionV3 model
    private let inceptionModel = Inceptionv3()
    
    // Create request variables
    private var requests = [VNCoreMLRequest]()
    
    // Create a live session
    let session = AVCaptureSession()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Run the function that transform the image into results
        createImageRequest()
        // Run function tat stat the live stream
        startLiveVideo()
    }
    //Function that transforms the image into results
    private func createImageRequest() {
        // create the visionModel
        guard let visionModel = try? VNCoreMLModel(for: self.inceptionModel.model) else {
            fatalError("problem creating request using core ml model")
        }
        // Create the visionRequest / visionModel as input 
        let visionRequest = VNCoreMLRequest(model: visionModel) {request, error in
            
            if error != nil {
                return
            }
            
            //Obtain request results
            guard let observations = request.results as? [VNClassificationObservation] else {
                return
            }
            // Map the request results
            let classifications = observations.map { observation in
                "\(observation.identifier) \(observation.confidence * 100.0)"
            }
            // Put the mapped results .identifier and .confidence in the textView container
            DispatchQueue.main.async {
                self.textView.text = classifications.joined(separator: "\n")
            }
            
            print(observations)
            
        }
        // Request : streaming of visionRequest
        self.requests.append(visionRequest)
    }
    
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else {
            return
        }
        
        var requestOptions:[VNImageOption : Any] = [:]
        
        if let camData = CMGetAttachment(sampleBuffer, kCMSampleBufferAttachmentKey_CameraIntrinsicMatrix, nil) {
            requestOptions = [.cameraIntrinsics:camData]
        }
        
        // Create a let imageRequestHandler that takes pixelBuffer as Input. 
        let imageRequestHandler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, orientation: CGImagePropertyOrientation(rawValue: 6)!, options: requestOptions)
        
        // imageRequestHandler executes requests
        //try imageRequestHandler.perform(self.requests)
        do {
            try imageRequestHandler.perform(self.requests)
        } catch {
            print(error)
        }
    }
    // Live stream function
    private func startLiveVideo() {
        
        //1
        session.sessionPreset = AVCaptureSession.Preset.photo
        let captureDevice = AVCaptureDevice.default(for: AVMediaType.video)
        
        //2 # The computation on your device not the simulator
        let deviceInput = try! AVCaptureDeviceInput(device: captureDevice!)
        let deviceOutput = AVCaptureVideoDataOutput()
        deviceOutput.videoSettings = [kCVPixelBufferPixelFormatTypeKey as String: Int(kCVPixelFormatType_32BGRA)]
        deviceOutput.setSampleBufferDelegate(self, queue: DispatchQueue.global(qos: DispatchQoS.QoSClass.default))
        session.addInput(deviceInput)
        session.addOutput(deviceOutput)
        
        //3
        let imageLayer = AVCaptureVideoPreviewLayer(session: session)
        imageLayer.frame = imageView.bounds
        imageView.layer.addSublayer(imageLayer)
        
        session.startRunning()
    }
    
}
```
