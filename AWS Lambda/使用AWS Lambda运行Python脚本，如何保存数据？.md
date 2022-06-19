# 使用AWS Lambda运行Python脚本，如何保存数据？

我有一个从API收集数据的脚本，在本地机器上手动运行这个脚本，我可以将数据保存到CSV或SQLite.db文件中。在

如果我把这个放在AWS lambda上，我如何存储和检索数据？

这实际上取决于你以后想如何处理这些信息。

- 如果您想将其保存在一个文件中，那么只需将其复制到amazons3。它可以存储任意多的数据。

- 如果要查询信息，可以选择将其放入数据库。根据您的需要，有许多不同的数据库选项可用。

**您可以在lambda函数的实例中保存数据，只是您不想将其用作永久存储。相反，您希望使用专门存储数据的云服务，这取决于您的用例**。

## *一些背景信息*

使用lambda时，您必须将其视为一个短暂的实例，在该实例中，您只能访问`/tmp`目录，并且最多可以保存512MB（[see lambda limits](https://docs.aws.amazon.com/lambda/latest/dg/limits.html)）。存储在`/tmp`目录中的数据可能仅在函数执行期间可用，并且不能保证保存在那里的任何信息在以后的执行中可用。

## *考虑事项*

这就是为什么您应该考虑使用其他云服务来存储数据，例如用于存储文件的简单存储服务（S3）、关系数据库的RDS或作为NoSQL数据库解决方案的DynamoDB。

还有许多其他选项，这都取决于用例。

## *工作溶液*

*对于python，使用boto3在S3中存储文件非常简单。代码使用库请求对谷歌并将输出保存到S3。作为附加步骤，它还会创建一个签名的URL，您可以使用它来下载文件*

```python
# lambda_function.py
import os
import boto3
from botocore.client import Config
import requests

s3 = boto3.resource('s3')
client = boto3.client('s3', config=Config(signature_version='s3v4'))

# This environment variable is set via the serverless.yml configuration
bucket = os.environ['FILES_BUCKET']

def lambda_handler(event, conntext):
    # Make the API CALL
    response = requests.get('https://google.com')

    # Get the data you care and transform it to the desire format
    body = response.text

    # Save it to local storage
    tmp_file_path = "/tmp/website.html"
    with open(tmp_file_path, "w") as file:
        file.write(body)
    s3.Bucket(bucket).upload_file(tmp_file_path, 'website.html')

    # OPTIONAL: Generar signed URL to download the file
    url = client.generate_presigned_url(
        ClientMethod='get_object',
        Params={
            'Bucket': bucket,
            'Key': 'website.html'
        },
        ExpiresIn=604800 # 7 days
    )
    return url
```

## *部署*

为了部署lambda函数，我强烈建议使用[Serverless](https://serverless.com/)或[LambdaSharp](https://github.com/LambdaSharp/LambdaSharpTool)之类的部署工具。下面是一个`serverless.yml`文件，用于serverless框架打包和部署代码，它还创建了S3存储桶，并设置了适当的权限来放置对象和生成签名的url：

*^{pr2}$*

现在打包并部署

````yaml
#!/usr/bin/env bash
# deploy.sh
mkdir package
pip install -r requirements.txt  target=./package
cp lambda_function.py package/
$(cd package; zip -r ../package.zip .)
serverless deploy  verbose
````

## 结论

运行lambda函数时，必须将它们视为无状态的。如果您想保存应用程序的状态，最好使用其他非常适合您的用例的云服务。

对于存储csv，S3是一个理想的解决方案，因为它是一个高可用性的存储系统，很容易开始使用python。

- 使用awslambda，您可以使用dynamo db之类的数据库，而不是sql数据库，并且可以从那里下载csv文件。

- 有了lambda到dynamo bd的集成非常容易，lambda是无服务器的，dynamo db是nosql数据库。

- 所以你可以把数据保存到dynamo数据库中，也可以使用RDS（Mysql）和man-other服务，但最好的方法是dynamo-db。

## 参照

https://www.cnpython.com/qa/197016