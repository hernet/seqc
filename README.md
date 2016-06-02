## SEquence Quality Control (SEQC -- /sek-si:/)

## Overview:

SEQC is a python package that processes single-cell sequencing data in the cloud and analyzes it interactively on your local machine.
 
To faciliate easy installation and use, we have made available Amazon Machine Images (AMIs) that come with all of SEQC's dependencies pre-installed. In addition, we have uploaded common genome indices (-i/--index parameter, see "Running SEQC) and barcode data (--barcode-files) to public amazon s3 repositories. These links can be provided to SEQC and it will automatically fetch them prior to initiating an analysis run. Finally, it will fetch data from BaseSpace or amazon s3 for analysis.

For users with access to in-house compute clusters, SEQC can be installed on your systems and run using the --local parameter.

### Dependencies:

#### Python3
<a href=https://www.python.org/downloads/>Python 3</a> must be installed on your local machine to run SEQC.

#### Amazon Web Services:
If you wish to run SEQC remotely on amazon (AWS), you must set up an AWS account (see below). SEQC will function on any machine running a Unix-based operating system (e.g. Google Compute Cloud), but we only provide AMIs for AWS.

#### Setting up an AWS Account: 
1. Navigate <a href=http://aws.amazon.com>here</a> and click “Create an AWS Account.”
2. Enter your desired login e-mail and click the “I am a new user” radio button.
3. Fill out the Login Credentials with your name and password.
4. Fill out your contact information, read the AWS Customer Agreement, and accept if you
wish to create an account.
5. Save your AWS Access ID and Secret Key -- this information is very important!

#### Create an RSA key to allow you to launch a cluster
1. Sign into your AWS account and go to the EC2 Dashboard.
2. Select the appropriate region (e.g. us-east-1 North Virginia)
2. Click “Key Pairs” in the NETWORK & SECURITY tab.
3. Click “Create Key Pair” and give it a new name.
4. This will install a new key called `<keyname>.pem` on your local machine. 
5. Rename the key to an .rsa extension and move it to a secure directory, e.g. `~/.ssh/<keyname>.rsa`. It is good practice to name the rsa key in an identifiable way (e.g. with your name) since it identifies which instances and volumes you own on the AWS dashboard. 
7. Change the permission settings with `$> chmod 600 /path/to/key/<keyname>.rsa`

#### Install and Configure AWS CLI (AWS Command Line Interface).
1. You can install by typing `pip3 install awscli`
2. Then, configure it by typing `aws configure`:
    * AWS Access Key ID [*******]: `access_id`
    * AWS Secret Access Key [*******]: `secret_key`
    * Default region name [us-west-2]: `us-east-1` (Adjust accordingly)
    * Default output format [None]: `text`

### Optional Dependencies

#### LibHDF5
<a href=https://www.hdfgroup.org/HDF5>libhdf5</a>, is highly efficient database used to store intermediate SEQC outputs. HDF5 can be easily installed with homebrew.

    $> brew install homebrew/science/hdf5


### SEQC Installation:

Once all dependencies have been installed, SEQC can be installed on any machine by typing:

    $> git clone https://github.com/ambrosejcarr/seqc.git
    $> pip3 install -e seqc/
    $> seqc/configure

Please follow the instructions of the configure script by providing SEQC information about your AWS access keys and, if you desire, a BASESPACE token. Once you have configured SEQC, you are ready to process single-cell sequencing data. Also note that when supplying the location of the RSA key during the configuration process, **the full path must be specified**.

### Running SEQC:

After SEQC is installed, help can be listed:

    $> process_experiment.py -h
    
    usage: process_experiment.py [-h] [--barcode-files [BF [BF ...]]] -o O -i I
                                 [-g [G [G ...]]] [-b [B [B ...]]] [-m [M]]
                                 [-s [S]] [-r [RA]] [--basespace BS]
                                 [--max-insert-size F] [--min-poly-t T]
                                 [--max-dust-score D] [--max-ed ED] [--local]
                                 [--aws] [--email-status E] [--no-terminate]
                                 [--check-progress] [-v]
                                 {in_drop,drop_seq,mars1_seq,mars2_seq,in_drop_v2}
    
    Process Single-Cell RNA Sequencing Data
    
    positional arguments:
      {in_drop,drop_seq,mars1_seq,mars2_seq,in_drop_v2}
                            which platform are you merging annotations from?
    
    optional arguments:
      -h, --help            show this help message and exit
      -v, --version         show program's version number and exit
    
    required arguments:
      --barcode-files [BF [BF ...]]
                            Either (a) an s3 link to a folder containing only
                            barcode files, or (b) the full file path of each file
                            on the local machine.
      -o O, --output-stem O
                            If remote=True (default), an s3 link to the directory
                            where all output data should be uploaded when the run
                            completes (e.g. s3://my-bucket/seqc_output_run_x/). If
                            remote=False, the complete path and prefix for the
                            output data (e.g. /Users/me/data/seqc/run_x_output).
                            Cannot have a terminal /.
      -i I, --index I       Local folder or s3 link to a directory containing the
                            STAR index used for alignment.
    
    input arguments:
      -g [G [G ...]], --genomic-fastq [G [G ...]]
                            List of fastq file(s) containing genomic information,
                            or an s3 link to a directory containing only genomic
                            fastq file(s).
      -b [B [B ...]], --barcode-fastq [B [B ...]]
                            List of fastq file(s) containing barcode information,
                            or an s3 link to a directory containing only barcode
                            fastq file(s).
      -m [M], --merged-fastq [M]
                            Filename or s3 link to a fastq file containing genomic
                            information annotated with barcode data.
      -s [S], --samfile [S]
                            Filename or s3 link to a .sam file containing aligned,
                            merged sequence records.
      -r [RA], --read-array [RA]
                            Filename or s3 link to a ReadArray (.h5) archive
                            containing processed sam records.
      --basespace BS        BaseSpace sample ID. The string of numbers indicating
                            the id of the BaseSpace sample. (e.g. if the link to
                            the sample
                            ishttps://basespace.illumina.com/sample/34000253/0309,
                            --basespace=34000253.
    
    filter arguments:
      --max-insert-size F   maximum paired-end insert size that is considered a
                            valid record. For multialignment correction. Not
                            currently used.
      --min-poly-t T        minimum size of poly-T tail that is required for a
                            barcode to be considered a valid record (default=3)
      --max-dust-score D    maximum complexity score for a read to be considered
                            valid. (default=10, higher scores indicate lower
                            complexity.)
      --max-ed ED           Maximum hamming distance for correcting barcode
                            errors. Should be set to 1 when running in_drop legacy
                            barcodes. (Default=2).
    
    run options:
      --local               Run SEQC locally instead of initiating on AWS EC2
                            servers.
      --aws                 Automatic flag; no need for user specification.
      --email-status E      Email address to receive run summary or errors when
                            running remotely.
      --no-terminate        Do not terminate the EC2 instance after program
                            completes.
      --check-progress      Check progress of all currently running remote SEQC
                            runs.

A typical run of a human "inDrop" sequencing experiment where data is downloaded from Illumina's BaseSpace proceeds as follows:

    $> process_experiment.py in_drop -i s3://<s3 genome location> -o s3://<aws upload location> --basespace <sample id> --barcode-files s3://<barcode file folder>/ 
