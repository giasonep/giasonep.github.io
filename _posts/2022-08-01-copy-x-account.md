---
title: "Copying Large Files Cross-Account Without Network Connectivity: A Data Engineering Challenge"
categories:
  - Blog
tags:
  - Data Engineering
  - Batch
  - Docker
  - Airflow
  - AWS
---
*Hello, data enthusiasts! It's been a while since my last blog post, and I'm thrilled to be back with an exciting project I worked on at my current company [Annalect](https://www.annalect.com/). For those who are new to [Annalect](https://www.annalect.com/), it is a leading data and analytics company that empowers marketers with data-driven insights to optimize their campaigns and strategies.*
{: .text-center}  

![image-center](/assets/images/copy_x_account/disconnected-pipeline.jpg){: .align-center}
<center><sup>Photo by <a href="https://unsplash.com/@manelandsean?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Manel & Sean</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></sup></center>  

In this blog post, I will share an interesting data engineering project I encountered early on in my career at [Annalect](https://www.annalect.com/). The project involved fixing a broken pipeline that required moving data from an old AWS S3 account source bucket to a new AWS S3 account target bucket for a new production process. The catch? Network connectivity between the accounts was not feasible due to backend restrictions.

## The Challenge: A Broken Pipeline

We had valuable client data that was delivered daily to an older AWS S3 account source bucket, which fed an existing and still-in-production data pipeline. My objective was to transfer this data to a new AWS S3 account target bucket, where a new production process would leverage it. However, I had to do it without network connectivity between the accounts.

## Initial Approach

Our initial attempt was to use the **S3FileTransformOperator()** along with `/bin/cp` for the **transform_scripts** argument, which served us well for most file transfers. To access the source AWS account, we used an AWS access key ID and secret access key stored as encrypted parameters in AWS Systems Manager. However, I soon encountered an issue - files larger than ~2 GB were failing due to the memory limitations in our *Airflow MWAA* environment.

## The New Process

To overcome the memory constraints, I devised a new process tailored for both small and large files. For files less than 1.5 GB in the source bucket, we continued to use the **S3FileTransformOperator()** for copying files to the target bucket, utilizing the *MWAA EC2* compute we were already paying for.

For files larger than 1.5 GB, I opted to kick off an *AWS Batch* Job, utilizing *EC2 Spot Instances* to reduce costs, for the transfer. I took care to provision more RAM for this process, and our *Docker Image*, stored on *AWS ECR*, contained a **python** script as its entrypoint which did all of the heavy lifting.

## Optimizing Large File Transfers

The real breakthrough in handling large files came with the implementation of the **boto3.s3.transfer** **TransferConfig()** Class. By setting the **max_concurrency** and enabling **threading** for multipart downloads and uploads, we significantly improved the performance of the data transfers. This was instrumental in ensuring efficient handling of large files, minimizing potential failures.

## Leveraging New Ideas

Another pivotal feature I utilized was the *Boto3* **Client** abstraction. Initially, our process involved manually adding new client files to a **config** in the ingestion pipeline, which proved cumbersome and inefficient. But, with the help of *Boto3's* **Client** abstraction, I listed the contents in the source bucket object path and used this list to copy all existing files, eliminating the need for manual intervention and ensuring seamless data transfers. 

## Key Technologies and Services:

The success of this data engineering pipeline relied on a powerful stack of technologies and services. We utilized *Docker* for containerization, *MWAA* for orchestration, *AWS Batch* for handling large files, and *Python* with *Boto3* for scripting and automation. The Docker Image was stored in *AWS ECR*, providing us with a scalable and reliable container registry.

## Conclusion

It's been nearly eight months since we deployed this enhanced data transfer pipeline, and I'm happy to report that it has been running smoothly without any issues. Our diligent approach to optimizing large file transfers and leveraging AWS services like *MWAA*, *AWS Batch*, and *Boto3* has proven effective in streamlining our data pipeline.

---

I hope you found this content interesting and I thank you for reading :bowtie:

**If you are a potential employer and interested in seeing some examples of what this code looks like, message me on my [Contact](https://giasonep.github.io/contact/) page or on [LinkedIn](https://linkedin.com/in/jasonmpalumbo) and I will happily walk you through my private repo on [GitHub](https://github.com/giasonep)! Cheers** :dizzy: