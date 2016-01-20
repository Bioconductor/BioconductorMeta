#### Location detailing Bioconductor AMIs
The Biocnoductor AMIs are listed here:
    http://www.bioconductor.org/help/bioconductor-cloud-ami/#ami_ids

#### How AMIs are created
An AMI (or _Amazon Machine Image_) is like a suspended clone of an EC2 instance.  
Since our AMI's include Rstudio, it's important to follow these steps before creating
your image :
```shell
# Stop the rstudio service
sudo service rstudio-server stop
# Since this doesn't stop all processes, find the remaining rstudio PID
ubuntu@ip-172-30-0-92:~$ sudo ps -ef |grep -i rstudio
sudo: unable to resolve host ip-172-30-0-92
ubuntu    1500     1  1 15:08 ?        00:00:00 /usr/lib/rstudio-server/bin/rsession -u ubuntu
ubuntu    1531  1476  0 15:09 pts/0    00:00:00 grep --color=auto -i rstudio
# Kill the rstudio rsession
kill 1500
# Remove the ~/.rstudio directory and the ~/.Rhistory file
rm -rf .rstudio/ .Rhistory
```
Next, shutdown the instance:
```
sudo shutdown -h now
```

Next, create an AMI by logging into the
[AWS console](https://bioconductor.signin.aws.amazon.com/console) .  Follow these steps :

1. Finding the EC2 instance that will serve as the basis for your image
2. Right click on the instance
3. Select Image > Create Image
4. A new Image (AMI) will be created.

If these steps become outdated, refer to the [AWS documentation](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html).

#### How AMIs are upgraded

#### Updating the R runtime

To update R on a Bioconductor AMI, perform the following operations :


\# FIXME: Fill content


#### Installing R packages

1. To install an R package from CRAN:
    ```
    # Example, installing the curl package
    biocLite("curl")
    ```

2. To install an R package from Bioconductor:
    ```
    # Example, installing the GenomicRanges package
    biocLite("GenomicRanges")
    ```

3. To install an R package from GitHub
    ```
    # Example, installing the Bioconductor/BiocIntroRPCI package
    # A slash indicates that this is a GitHub <account>/<repository> pair
    biocLite("Bioconductor/BiocIntroRPCI")
    ```

#### Installing R packages with Vignettes

1. To install an R package from GitHub and build it's vignettes
```
# Example, installing the curl package
biocLite("Bioconductor/BiocIntroRPCI", build_vignettes=TRUE)
```


To update R on a Bioconductor AMI, perform the following operations :


#### Updating R packages

To update R on a Bioconductor AMI, perform the following operations :

\# FIXME: Fill content


## User accounts
\# FIXME: Which user accounts will be used when a launching `R`?  What about installing packages?
