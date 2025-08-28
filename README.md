Client Project Brief


Scenario Overview
Client: James Smith, freelance web designer

Project: Portfolio Website Deployment

Project Description:
James Smith has designed a modern, responsive single-page portfolio website using Next.js. He needs this website hosted on a robust, scalable, and cost-effective platform with global availability and fast loading times.

Your Role:
As cloud engineers, you will deploy James's Next.js portfolio website on AWS using Infrastructure as Code (IaC) principles with Terraform.

Requirements
The website must be:

Highly Available: Accessible worldwide with minimal downtime

Scalable: Able to handle increasing traffic without performance degradation

Cost-Effective: Optimized hosting costs without unnecessary expenses

Fast Loading: Quick loading times for all visitors globally

Project Objectives
By completing this project, you will:

Deploy a Next.js website on AWS

Implement Infrastructure as Code using Terraform

Configure global content delivery with AWS CloudFront

Apply security and performance best practices

Host all project files and code on GitHub

Understanding Next.js
What is Next.js?
Next.js is an open-source React framework maintained by Vercel that enhances the development experience for building web applications.

Key Benefits
Server-Side Rendering (SSR): Generates pages on the server for each request

Static Site Generation (SSG): Pre-renders pages at build time

API Routes: Built-in API routing system for serverless functions

File-Based Routing: Simplified navigation through file structure

Built-In CSS and Sass Support: Easy styling implementation

Automatic Code Splitting: Loads only necessary JavaScript for current page

Common Use Cases
Static Websites: Blogs, landing pages, and portfolio sites

E-Commerce Sites: Fast-loading product pages optimized for SEO

Corporate Websites: Scalable sites for handling significant traffic

Web Applications: From simple dashboards to complex web apps

Blogs and Content Sites: Easy-to-maintain, SEO-friendly content platforms

Step 1: Prepare the Next.js Application
1.1 Create a GitHub Repository
Go to GitHub and create a new repository named terraform-portfolio-project

Initialize the repository with a README file

Clone the repository to your local machine: 

git clone https://github.com/<your-username>/terraform-portfolio-project.gitcd terraform-portfolio-project
1.2 Clone the Next.js Portfolio Starter Kit
Clone the Portfolio Starter Kit:

npx create-next-app@latest nextjs-blog --use-npm --example "https://github.com/vercel/next-learn/tree/main/basics/learn-starter"
Navigate to the project directory and start the development server:

cd blog
npm run dev
Access your Next.js application at http://localhost:3000/




Create Configuration File
In the root of your project folder, create a new file called next.config.js

Paste the following code into the file:

/**
 * @type {import('next').NextConfig}
 */
const nextConfig = {
  output: 'export',
}

module.exports = nextConfig
Build Your Project
After setting up the configuration, run the build command:

npm run build
This will generate a static export of your Next.js application in the out directory, which can be deployed to any static hosting service.



 

Step 1: Initialize Git and Push to GitHub
git init
git add .
git commit -m "Initial commit of Next.js portfolio starter kit"
git remote add origin https://github.com/<your-username>/terraform-portfolio-project.git
git push -u origin master

Step 2: Set Up Terraform Configuration
Create Project Directory

mkdir terraform-nextjs
cd terraform-nextjs
Create Terraform Files

a. State File with S3 + DynamoDB

b. Main.tf with AWS Provider Block

c. S3 Bucket Resource

resource "aws_s3_bucket" "website" {
  bucket = "your-unique-bucket-name"
  
  website {
    index_document = "index.html"
    error_document = "index.html"
  }
  
  tags = {
    Name = "Portfolio Website"
    Environment = "Production"
  }
}

d. S3 Bucket Policy Resource

resource "aws_s3_bucket_policy" "website_policy" {
  bucket = aws_s3_bucket.website.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = "*"
        Action = "s3:GetObject"
        Resource = "${aws_s3_bucket.website.arn}/*"
      }
    ]
  })
}

e. CloudFront Distribution

resource "aws_cloudfront_distribution" "website_distribution" {
  origin {
    domain_name = aws_s3_bucket.website.website_endpoint
    origin_id   = "S3-Website"
    
    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "http-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }
  
  enabled             = true
  default_root_object = "index.html"
  
  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-Website"
    
    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
    
    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }
  
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
  
  viewer_certificate {
    cloudfront_default_certificate = true
  }
  
  tags = {
    Name = "Portfolio CloudFront"
    Environment = "Production"
  }
}

f. outputs.tf File

output "bucket_website_endpoint" {
  value = aws_s3_bucket.website.website_endpoint
}

output "cloudfront_url" {
  value = aws_cloudfront_distribution.website_distribution.domain_name
}
Initialize and Apply Terraform

terraform init
terraform plan
# Review for any issues and resolve if needed
terraform apply
Step 3: Upload the Next.js Static Site to S3
aws s3 sync ../blog/out s3://your-bucket-name
Note: Make sure you're using the correct path to your Next.js build output folder and the correct S3 bucket name.

Step 4: Access the Deployed Website
Get the CloudFront URL

terraform output cloudfront_url
Access Your Website Open the CloudFront URL in your web browser to view your deployed Next.js portfolio site.

<img width="1911" height="1024" alt="image" src="https://github.com/user-attachments/assets/15cd0b8a-8c94-47e9-adf6-fee18be7699a" />

