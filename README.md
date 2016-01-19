# Lee-iphone-camera
使用系统相机

#import "ViewController.h"
#import <AVFoundation/AVFoundation.h>

@interface ViewController ()<UIImagePickerControllerDelegate,UINavigationControllerDelegate>
@property (nonatomic, weak) UIImageView *imageView;
@property (nonatomic, weak) UIButton *tackPhotoBtn;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    UIImageView *imageView = [[UIImageView alloc] init];
    imageView.backgroundColor = [UIColor redColor];
    imageView.contentMode = UIViewContentModeScaleAspectFit;
    imageView.frame = CGRectMake(20, 20, 320, 320);
    [self.view addSubview:imageView];
    _imageView = imageView;
    
    UIButton *tackPhotoBtn = [UIButton buttonWithType:UIButtonTypeCustom];
    [tackPhotoBtn setTitle:@"take photo" forState:UIControlStateNormal];
    tackPhotoBtn.frame = CGRectMake(20, 440, 30, 40);
    tackPhotoBtn.backgroundColor = [UIColor grayColor];
    [tackPhotoBtn addTarget:self action:@selector(takePhoto:) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:tackPhotoBtn];
    _tackPhotoBtn = tackPhotoBtn;
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

#pragma mark -  Action

- (void)takePhoto:(UIButton *)btn
{
    // 1. 是否有相机
    if (![UIImagePickerController isSourceTypeAvailable: UIImagePickerControllerSourceTypeCamera]) {
        NSLog(@"无相机功能");
        return;
    } else {
        // 2. 获取相机访问权限
        NSString *mediaType = AVMediaTypeVideo;
        AVAuthorizationStatus authStatus = [AVCaptureDevice authorizationStatusForMediaType:mediaType];
        
        if(authStatus ==AVAuthorizationStatusRestricted){
            // 2.1 受限制
            NSLog(@"Restricted");
        }else if(authStatus == AVAuthorizationStatusDenied){
            // 2.2 不允许访问
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"提示"
                                                            message:@"请在设备的 设置-隐私-相机 中允许访问相机。"
                                                           delegate:self
                                                  cancelButtonTitle:@"确定"
                                                  otherButtonTitles:nil];
            [alert show];
            return;
        }
        else if(authStatus == AVAuthorizationStatusAuthorized){
            // 2.3 调取相机
            UIImagePickerController *pickerImage = [[UIImagePickerController alloc] init];
            if([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
                pickerImage.sourceType = UIImagePickerControllerSourceTypeCamera;
                //                pickerImage.mediaTypes = [UIImagePickerController availableMediaTypesForSourceType:pickerImage.sourceType];
            }
            pickerImage.delegate = self;
            pickerImage.allowsEditing = NO;
            [self presentViewController:pickerImage animated:YES completion:^{}];
            
        }else if(authStatus == AVAuthorizationStatusNotDetermined){
            // 2.4 未获取授权
            [AVCaptureDevice requestAccessForMediaType:mediaType completionHandler:^(BOOL granted) {
                if(granted){//点击允许访问时调用
                    // 2.4.1 调取相机
                    UIImagePickerController *pickerImage = [[UIImagePickerController alloc] init];
                    if([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypePhotoLibrary]) {
                        pickerImage.sourceType = UIImagePickerControllerSourceTypeCamera;
                        //                        pickerImage.mediaTypes = [UIImagePickerController availableMediaTypesForSourceType:pickerImage.sourceType];
                        
                    }
                    pickerImage.delegate = self;
                    pickerImage.allowsEditing = NO;
                    [self presentViewController:pickerImage animated:YES completion:^{}];
                    
                }
                else {
                    // 2.4.2 未获取相机访问授权
                }
                
            }];
        }else {
            // 1.4未知状态
        }
    }
    
}

#pragma mark - UIImagePickerControllerDelegate
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info
{
    
    UIImage *image = [info objectForKey:UIImagePickerControllerOriginalImage];
    self.imageView.image = image;

    [picker dismissViewControllerAnimated:YES completion:^{
        
    }];
}
