# ServerCall Delegate Method



import Foundation
import Alamofire

let DEVICE_TYPE = "1"

let URL_USER_PRofile                     = BasePath + BasePathServerURl + "update_teacher"
let URL_TEACHER_SUBJECT                  = BasePath + BasePathServerURl + "get_class_by_subject_teacher"
let GET_ADD_PHOTO_GELLERY_IMAGES         = BasePath + BasePathServerURl + "photo_gallery_add_images"
let GET_CREATE_ASSIGNMENT                = BasePath + BasePathServerURl + "create_assignment_by_teacher"
let GET_UPDATE_PROFILE                   = BasePath + BasePathServerURl + "update_profile"
let STUDENT_UPLOAD_CERTIFICATE           = BasePath + BasePathServerURl + "student_upload_ducument"
let GET_SUBMIT_ASSIGMENTDATA           = BasePath + BasePathServerURl + "student_submit_assignment"

enum HTTP_METHOD {
    case http_GET, http_POST
}

enum ServerCallName : Int {
    case URL_USER_PROFILE = 101,
    URL_TEACHER,imageUplad,
    CreateAssigment,
    UpdateProfile,
    student_upload_ducument,
    Get_sumit_assign
    //GET_SUBMIT_ASSIGMENT
    
}
protocol ServerCallDelegate {
    func ServerCallSuccess(_ resposeObject: AnyObject, name: ServerCallName)
    func ServerCallFailed(_ errorObject:String, name: ServerCallName)
}
extension UIImage {
    // Loads image asynchronously
    class func loadFromPath(_ path: String, callback:@escaping (UIImage) -> Void ) {
        Alamofire.request(path).responseData { (theResponse : DataResponse<Data>) in
            if let imgData = theResponse.data {
                if let img = UIImage(data: imgData) {
                    callback(img)
                }
                else {
                    callback(UIImage())
                }
            }
            else {
                callback(UIImage())
            }
        }
    }
}
class ServerCall: NSObject {
    var delegateObject : ServerCallDelegate!
    
    // Shared Object Creation
    static let sharedInstance = ServerCall()

    // Make API Request WITH Header
    func requestWithUrlAndHeader(_ httpMethod: HTTP_METHOD, urlString: String, header: [String : String], delegate: ServerCallDelegate, name: ServerCallName) {
        self.delegateObject = delegate
        let methodOfRequest : HTTPMethod = (httpMethod == HTTP_METHOD.http_GET) ? HTTPMethod.get : HTTPMethod.post
        
        let queue = DispatchQueue(label: "com.Technothum.manager-response-queue", attributes: DispatchQueue.Attributes.concurrent)
        
        let request = Alamofire.request(urlString, method: methodOfRequest, parameters: nil, encoding: JSONEncoding.default, headers: header)
        
        request.responseJSON(queue: queue,options: JSONSerialization.ReadingOptions.allowFragments)
        {
            (response : DataResponse<Any>)
            in DispatchQueue.main.async
                {
                    print("Am I back on the main thread: \(Thread.isMainThread)")
                    if (response.result.isSuccess) {
                        self.delegateObject.ServerCallSuccess(response.result.value! as AnyObject, name: name)
                    }
                    else if (response.result.isFailure) {
                        self.delegateObject.ServerCallFailed((response.result.error?.localizedDescription)!, name: name)
                    }
            }
        }
    }
    // Make API Request WITH Parameters
    func requestWithUrlAndParameters(_ httpMethod: HTTP_METHOD, urlString: String, parameters: [String : AnyObject], delegate: ServerCallDelegate, name: ServerCallName) {
        
        
        self.delegateObject = delegate
        let methodOfRequest : HTTPMethod = HTTPMethod.post
        
        let queue = DispatchQueue(label: "com.Technothum.manager-response-queue", attributes: DispatchQueue.Attributes.concurrent)
        
        _ = URL(string: urlString)
        
        let request = Alamofire.request(urlString, method: methodOfRequest, parameters: parameters)
        request.responseJSON(queue: queue,
                             options: JSONSerialization.ReadingOptions.allowFragments) {
                                (response : DataResponse<Any>) in
                            
                                DispatchQueue.main.async {
                                    //  print("Am I back on the main thread: \(Thread.isMainThread)")
                                    if (response.result.isSuccess) {
                                        self.delegateObject.ServerCallSuccess(response.result.value! as AnyObject, name: name)
                                    }
                                    else if (response.result.isFailure) {
                                        self.delegateObject.ServerCallFailed((response.result.error?.localizedDescription)!, name: name)
                                    }
                                }
        }
    }
    //MARK:- Multi part request with parameters.
    func requestMultiPartWithUrlAndParameters(_ urlString: String, parameters: [String : AnyObject], image: UIImage, imageParameterName: String, imageFileName:String, delegate: ServerCallDelegate, name: ServerCallName) {
        
        self.delegateObject = delegate
        var imageData : Data!
        imageData = UIImagePNGRepresentation(image)
        
        Alamofire.upload(
            multipartFormData: { multipartFormData in
                // The for loop is to append all parameters to multipart form data.
                
                if imageData != nil{
                    multipartFormData.append(imageData, withName: imageParameterName, fileName: imageFileName + ".png", mimeType: "image/png")
                }
                
                
                for element in parameters.keys{
                    let strElement = String(element)
                    let strValueElement = parameters[strElement] as! String
                    multipartFormData.append(strValueElement.data(using: String.Encoding.utf8, allowLossyConversion: false)!, withName: strElement)
                }
                
                // Append Image to multipart form data.
//                multipartFormData.append(imageData, withName: imageParameterName, fileName: imageFileName + ".png", mimeType: "image/png")
        },
            to: urlString,
            encodingCompletion: { encodingResult in
                switch encodingResult {
                case .success(let upload, _, _):
                    upload.responseJSON { response in
                        debugPrint(response)
                        //                        print(response.result.value!)
                        //                        print(response.data as! NSData)
                        self.delegateObject.ServerCallSuccess(response.result.value as AnyObject, name: name)
                    }
                case .failure(let encodingError):
                    print(encodingError)
                    self.delegateObject.ServerCallFailed(encodingError.localizedDescription, name: name)
                }
        })
    }
   
    
    //User Register
//    func UserRegister(endUrl: String, profileImage: UIImage,parameter:[String: Any]?, onCompletion:(([String:AnyObject]) -> Void)? = nil, onError: ((Error?) -> Void)? = nil){
//
//        let manager = Alamofire.SessionManager.default
//        manager.session.configuration.timeoutIntervalForRequest = 300
//        manager.upload( multipartFormData: { multipartFormData in
//
//            if let imageData = UIImageJPEGRepresentation((profileImage), 0.7) {
//                multipartFormData.append(imageData, withName: "keyimage", fileName: "image.png", mimeType: "image/png")
//            }
//
//            for (key, value) in parameter! {
//                multipartFormData.append("\(value)".data(using: String.Encoding.utf8)!, withName: key as String)
//            }
//
//        },
//
//                        to: endUrl,
//                        encodingCompletion: { encodingResult in
//
//                            switch encodingResult {
//                            case .success(let upload, , ): upload.responseJSON { response in
//
//                                let data = response.data!
//                                var dictonary = [String:AnyObject]()
//                                do {
//                                    dictonary =  (try JSONSerialization.jsonObject(with: data, options: .allowFragments) as? [String:AnyObject])!
//                                    onCompletion!(dictonary as [String : AnyObject])
//
//                                } catch let error as NSError {
//                                    onError?(error)
//                                }
//
//                                }.uploadProgress { progress in // main queue by default
//                            }
//                                return
//
//                            case .failure(let encodingError):
//                                onError?(encodingError)
//                            }
//
//        })
//
//    }
    
}

/// Api Call Edit Profile Demo

import UIKit
import SVProgressHUD
import Alamofire
import SDWebImage
class EditProfileVC: UIViewController,UIImagePickerControllerDelegate,UINavigationControllerDelegate,ServerCallDelegate {
    
    @IBOutlet var btnProfile: UIButton!
    @IBOutlet var txtFirstName: UITextField!
    @IBOutlet var txtLastName: UITextField!
    @IBOutlet var txtEmail: UITextField!
    var FistName = String()
    var LastName = String()
    var Email = String()
    var ProfileImage = String()
    private var actionSheet = UIAlertController()
    private var cameraController = UIImagePickerController()
    var selectedImage = UIImage()
    
    //Williams
    override func viewDidLoad() {
        super.viewDidLoad()
        self.txtEmail.text = Email
        
        if FistName != "<null>"{
            self.txtFirstName.text = FistName
            
        }
        if LastName != "<null>"{
            self.txtLastName.text = LastName
        }
//       self.btnProfile.sd_setImage(with:URL(string:ProfileImage) , for: .normal, completed: nil)
       
//        self.btnProfile.sd_setImage(with:URL(string:ProfileImage), for: UIImage(named: "placeholder"))
//        btnProfile.sd_setImage(with: URL(string:ProfileImage), for:.normal, placeholderImage:btn, completed: nil)
        self.btnProfile.sd_setBackgroundImage(with: URL(string:ProfileImage), for: UIControl.State.normal, placeholderImage: UIImage(named: "ic_userPhoto"))

    }
    //Validetion
    private func validatePRofile() -> Bool {
        var msg : String? = nil
        if (txtFirstName.text?.trim().isEmpty == true) {
            msg = "Please Enter \(txtFirstName.placeholder!)"
        }
        else if (txtLastName.text?.trim().isEmpty == true) {
            msg = "Please Enter \(txtLastName.placeholder!)"
        }
       
        if msg != nil {
            AlertViewUtli.showAlertFromController(controller:self, withMessage: msg!)
            return false
        }
        return true
    }
    @IBAction func TapToUpdate(_ sender: Any) {
        if validatePRofile() == false{
            return
        }
        self.Get_Update_Profile()
    }
    @IBAction func TapToBack(_ sender: UIButton) {
        self.navigationController?.popViewController(animated: true)
    }
    @IBAction func TapToPRofile(_ sender: UIButton) {
        let actionSheet = UIAlertController(title: "Rarom", message: "choose as you like here!", preferredStyle: .actionSheet)
        
        actionSheet.addAction(UIAlertAction(title: "Photo Library", style: .default, handler: {
            action in
            _ = self.startCameraFromViewController(self, sourceType:.photoLibrary, withDelegate:self)
        }))
        actionSheet.addAction(UIAlertAction(title: "Choose From Camera Roll", style: .default, handler: {
            action in
            _ = self.startCameraFromViewController(self, sourceType: .camera, withDelegate: self )
        }))
        
        actionSheet.addAction(UIAlertAction(title: "Cancel", style: .destructive, handler: {
            action in
            
        }))
        actionSheet.popoverPresentationController?.sourceView = self.view
        actionSheet.popoverPresentationController?.sourceRect = sender.frame
        present(actionSheet, animated: true, completion: nil)
    }
    //MARK :- Profile Imageset
    func startCameraFromViewController(_ viewController: UIViewController,sourceType:UIImagePickerControllerSourceType, withDelegate delegate: UIImagePickerControllerDelegate & UINavigationControllerDelegate) -> Bool {
        
        if UIImagePickerController.isSourceTypeAvailable(sourceType) == false {
            return false
        }
        cameraController = UIImagePickerController()
        cameraController.sourceType = sourceType
        cameraController.allowsEditing = true
        cameraController.delegate = delegate
        
        present(cameraController, animated: true, completion: nil)
        return true
    }
    func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [String : Any]) {
        
        
        picker.dismiss(animated: true) {
           
            if let image = info[UIImagePickerControllerEditedImage] as? UIImage {
                self.selectedImage = self.selectedImage.resizeImage(image: image, newSize: CGSize(width: 68, height: 68))
                self.btnProfile.setImage(self.selectedImage, for:.normal)
            }
        }
        
    }
    func Get_Update_Profile(){
        
        SVProgressHUD.show(withStatus: "Loading...")
        SVProgressHUD.setDefaultMaskType(.black)
        Pref.setObject(txtFirstName.text! + " " + txtLastName.text!, forKey: Kname)
        let ShcoolID = Pref.getObjectForKey(Kschool_id)as! String
        let RanningYear = Pref.getObjectForKey(Krunning_year)as! String
        let USerID = Pref.getObjectForKey(Kuser_id)as! Int
        let USerType = Pref.getObjectForKey(Kuser_type)as! String
        
        
        if USerType == "school_admin"{
            let Param = ["user_id":"\(USerID)",
                "school_id":"\(ShcoolID)",
                "running_year":"\(RanningYear)",
                "user_type":"\(USerType)",
                "teacher_id":"\(USerID)",
                "first_name":"\(txtFirstName.text!)",
                "last_name":"\(txtLastName.text!)",
                "email":"\(txtEmail.text!)"]
            print(Param)
            print(Param)
            ServerCall.sharedInstance.requestMultiPartWithUrlAndParameters(GET_UPDATE_PROFILE, parameters: Param as [String : AnyObject], image: self.selectedImage, imageParameterName: "userfile", imageFileName: "MyProfile", delegate: self, name: .UpdateProfile)


        }else{
            let Param = ["user_id":"\(USerID)",
                "school_id":"\(ShcoolID)",
                "running_year":"\(RanningYear)",
                "user_type":"\(USerType)",
                "teacher_id":"\(USerID)",
                "name":"\(txtFirstName.text!)",
                "last_name":"\(txtLastName.text!)",
                "email":"\(txtEmail.text!)"]
            print(Param)

            print(Param)
            ServerCall.sharedInstance.requestMultiPartWithUrlAndParameters(GET_UPDATE_PROFILE, parameters: Param as [String : AnyObject], image: self.selectedImage, imageParameterName: "userfile", imageFileName: "MyProfile", delegate: self, name: .UpdateProfile)
        }

        
        
        
    }
    
    func ServerCallSuccess(_ resposeObject: AnyObject, name: ServerCallName) {
        SVProgressHUD.dismiss()
        
        if name == .UpdateProfile {
//            print("\n\n\n Update Teacher SUCCESS ........... \n \(resposeObject)")
            print(resposeObject)
            let ResponceData = resposeObject as? [String:AnyObject]
            let States = TO_INT(ResponceData?["status"])
            let Message = TO_STRING(ResponceData?["message"])
            
            if States == 1{
                
                self.navigationController?.popViewController(animated: true)
                appDelegate.window?.rootViewController?.view.makeToast(message: "\(String(describing: "Sucessfully Profile Update"))")
            }else{
                appDelegate.window?.rootViewController?.view.makeToast(message: "\(String(describing: Message))")
            }
            
        }
    }
    func ServerCallFailed(_ errorObject: String, name: ServerCallName) {
        SVProgressHUD.dismiss()
        print("\n\n\n Update Teacher Faile ........... \n \(errorObject)")
        
    }
   
    
}








