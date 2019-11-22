# Module 3: On-ride photo processing - Creating the chroma key Lambda function

*[Click here](../README.md) to return to the main instructions for Module 3 at any time.*

This function implements [chroma key processing](https://en.wikipedia.org/wiki/Chroma_key), also commonly known as green screen. It takes an input image of a person against a green background, removes the green, and then saves the output image.

## Inside this section

Lambda functions can be written in different runtimes and can also use pre-packaged libraries of code called Lambda Layers.

- This section shows how you can use different runtimes for different tasks. The chromakey processing function uses an open source Python library called OpenCV. This function is deployed using Python 3.6 while the other functions are written in Node.
- The OpenCV library must be compiled using the target operating system, which for Lambda is Amazon Linux 2. To simplify deployment, this was already created in Module 1 and has been packaged as a Lambda Layer. You will use link this Layer to your function.

## Creating the Chromakey Lambda function

**:white_check_mark: Step-by-step Instructions**

1. Go to the Lambda console - from the AWS Management Console, select **Services** then select **Lambda** under *Compute*. **Make sure your region is correct.**

2. Select **Create function**. Enter `theme-park-photos-chromakey` for *Function name* and ensure `Python 3.6` is selected under *Runtime*.

3. Open the *Choose or create an execution role* section:
-  Select the *Use an existing role* radio button. 
- Click the *Existing role* drop-down, and enter **ThemeParkLambdaRole** until the filter matches a single available role beginning with *theme-park-backend-ThemeParkLambdaRole**. 
- Select this role.
- Select **Create function**.

![Module 3 - Create Function](../../images/3-photos-chroma1.png)

1. Select **+ Add Trigger**:
   - In the *Trigger configuration* dropdown, select **S3**. 
   - In the Bucket dropdown, select the bucket name beginning with `theme-park-backend-uploadbucket`. 
   - For *Event Type* select **All object create events** from the dropdown. 
   - Leave **Enable trigger** checked, and select **Add**.

![Module 3 - Add trigger](../../images/3-photos-chroma7.png)

2. In the *Designer* card, under the lambda function name `theme-park-photos-chromakey` select the **Layers** button to open the *Layers* panel below.

![Module 3 - Trigger added](../../images/3-photos-chroma2.png)

3. Select **Add a layer**. Select the *Provide a layer version ARN* radio button. Depending on the selected region, copy the ARN to the clipboard: 

- us-west-2 (US West - Oregon) - **arn:aws:lambda:us-west-2:678705476278:layer:Chromakey:1**
- us-east-2 (US East - Ohio) - **arn:aws:lambda:us-east-2:678705476278:layer:Chromakey:1**
- us-east-1 (US East - Northern Virginia) - **arn:aws:lambda:us-east-1:678705476278:layer:Chromakey:1**

Paste the ARN into the *Layer version ARN* field. Select **Add**.

![Module 3 - Add layer](../../images/3-photos-chroma3.png)

4. In the *Designer* card, select the `theme-park-photos-chromakey` Lambda function name to open the *Function code* panel below.

![Module 3 - Open code panel](../../images/3-photos-chroma4.png)

5. Copy the code from the file in Cloud9 by navigating to `3-photos/1-chromakey/app.py` onto the clipboard and paste into the `lambda_function.py` tab in the Lambda function:

![Module 3 - Paste code](../../images/3-photos-chroma5.png)

6. **Select *Save* to save the changes**.

### Adding environment variables

This function uses three environment variables:
- OUTPUT_BUCKET_NAME: the name of the bucket where the output object is stored.
- HSV_LOWER: A tuple representing lower HSV value for the chroma key matching process.
- HSV_UPPER: A tuple representing upper HSV value for the chroma key matching process.

In this section, you will retrieve and configure these Environment Variables for the function.

**:white_check_mark: Step-by-step Instructions**

1. Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select **Services** then select **Cloud9** under *Developer Tools*. **Make sure your region is correct.**

2. In the terminal enter the following command to retrieve the value for OUTPUT_BUCKET_NAME:
```
aws s3 ls | grep theme-park-backend-processingbucket
```

3. Go back to the browser tab with the `theme-park-photos-chromakey` Lambda function open. Scroll down to the *Environment variables* card and enter the three environment variables with the three variables, as follows:

- OUTPUT_BUCKET_NAME: the value from step 2 above.
- HSV_LOWER: (36, 100, 100)
- HSV_UPPER: (70 ,255, 255)

![Module 3 - Environment vars](../../images/3-photos-chroma6.png)

4. **Select *Save* to save these changes.**

### Change settings for the Lambda function

In this section, you will modify the memory and timeout settings for the Lambda function. This will proportionally also add more CPU and network resources to the lambda function.

**:white_check_mark: Step-by-step Instructions**

1. In the browser tab with the `theme-photos-chromakey` Lambda function open, scroll down to the *Basic settings* card.

![Module 3 - Basic settings](../../images/3-photos-chroma11.png)

2. Change the *Memory (MB)* slider to **3008 MB**. Change the *Timeout* values to **0** min and **10** sec. 

3. Select **Save** to update the function.

## Test the function

You will now test the function using a test image containing a photo of a person against a green background. You will manually copy this image into the upload bucket, and see the result in the processing bucket.

**:white_check_mark: Step-by-step Instructions**

1. Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select **Services** then select **Cloud9** under *Developer Tools*. **Make sure your region is correct.**

2. Navigate to the file `aws-serverless-workshop-innovator-island\3-photos\green-screen-png` and open. You can see the photo of a person with a green screen. This is the local testing image.

3. In the terminal enter the following command to change the directory:

```
cd ~/environment/aws-serverless-workshop-innovator-island/3-photos/
```
4. Find the name of your S3 upload bucket with this command:
```
aws s3 ls | grep uploadbucket
```
5. Copy the local testing image to your upload bucket, replacing the `youruploadbucket` bucket parameter with your bucket name from step 3:
```
aws s3 cp ./green-screen-test.png s3://youruploadbucketname
```
6. In another browser tab, open the AWS Console's **S3 console**.

![Module 3 - S3](../../images/3-photos-chroma12.png)

6. Select the `theme-park-backend-processingbucket`.

![Module 3 - S3](../../images/3-photos-chroma14.png)

7. Check the `green-screen-test.png` object, then select **Download**.

8. Save the file locally and open in an image viewer. 

9. You will see the original green screen image has been modified showing the person with the green background now removed. The Lambda function has been invoked when the photo was uploaded to the S3 bucket. The function ran a chromakey process using a library imported using a Lambda Layer which removed the green screen and then wrote the resulting image to another S3 bucket.

## Next steps

Next, you will create the compositing Lambda function. To start the next section, [click here to continue](../2-compositing/README.md).
