# 2Taps API Bridge
Dependencies

    nodejs 8.9.4 windows x64 :: https://nodejs.org/download/release/v8.9.4/node-v8.9.4-x64.msi
    aws-iot-device-sdk@2.2.0
    auto-updater@1.0.0

    AWS IOT IAM rule engine role: restpay_{$env}_pc_tasks_to_dynamodb
    AWS Dynamodb table restpay_{$env}_pc_tasks
        partition key id_user (string)
        sort key timestamp (string)
    AWS IOT rule engine rule: restpay_{$env}_pc_publish
        Query statement: SELECT * FROM 'restpay_{$env}_pc_publish'
        Action: Insert a message into a DynamoDB table -> restpay_{$env}_pc_tasks
            Hashkey: id_user
            Hashkey value: ${id_user}                   (string)
            Range key: timestamp
            Range key value : ${timestamp}              (string)
            Write message data to this column: Payload
    

Instalation on AWS

    ------- (ONLY IF NOT ALREADY DONE) ------- 

    - Create Policy (ONLY IF NOT ALREADY DONE)
        - Go to Security and select option “Create”
        - Set the name as restpay_{$env}_pcs_iot_policy
        - In Add Statements > Actions, select iot.*
        - In Resource ARN, just add “*”. Then Press “Create”

    ------- (ONLY IF NOT ALREADY DONE) ------- 

    ------- (ONLY IF NOT ALREADY DONE FOR RESTAURANT PC) ------- 

    - Create a certificate for the restaurant pc if not already generated
        - Go to AWS IoT console and go to security then certificates 
        - Press Create button located right hand top side. It will travels through "Create A certificate ” screen, select create certificate option
        - Download private key and certificate (not root certificate)
            - A private key: downloaded the file xxxxxxxxx-private.pem.key
            - A certificate for the thing: downloaded the file xxxxxxxxxx-certificate.pem
        - IMPORTANT : Go back to security / certificates page and activate the certificated

    - Create the restaurant pc if not already created (Thing in aws naming)
        - Go to Manage, Select things. 
        - Press Create Button from Top right. 
        - Give a name restpay_{$env}_pc_{$id_seller} <= this is a variable get from the main database on table seller)
        - And press “Create Thing”. 

    - Linking thing, Certificate and Policy
        - Go to Security > Certificate and open the certificate which we have created. 
        - Select Policy sub menu and press Actions button located in top right side of security tab. 
        - Select “Attach Policy” and select the created Policy. 
        - Then Select “Attach Thing” and select the created Thing

    ------- (ONLY IF NOT ALREADY DONE FOR RESTAURANT PC) ------- 

Instalation on restaurant Windows PC

    - Download and install nodejs 8.9.4 windows
    - Download https://github.com/GuilhermeMoura1/RestpayBridge/archive/master.zip
    - Extract to C:\2Taps\RestpayBrige
    - Create a file C:\2Taps\RestpayBridge\config.json following the config.json.dist example/structure
    - In C:\2Taps\RestpayBrige\credentials
        - Paste private key and certificate obtained from aws iot
    - Open Windows Cmd as Administrator
    - run C:\2Taps\RestpayBrige\install.cmd
        - The last step of this command is to install main.js as a window service
        - It will prompt you to give some permissions, click yes until no more windows apear asking permissions

Auto Updater

    IMPORTANT: 
        When you are ready to deploy the update
        You must update package.json with new version otherwise the auto updater will do nothing

    There are two methods that the app is updated
        1 - When the file main.js is started
        2 - When you publish any message to the AWS IOT Topic: restpay_{$env}_pc_update
            IMPORTANT: the auto update relies on package.json version attribute and it takes some minutes for git to update the file after commit
            Better to wait like 10 minutes
            
    CAREFULL: 
        The auto updater will stop the app and may lose incoming/processing messages
        Better to release the updates at morning start
        Take extra care with the update method 2 above