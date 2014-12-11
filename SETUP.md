###Assuming python3 is not on your server and you are not sudo

1. Download python source, compile  
```
wget http://python.org/ftp/python/3.3.4/Python-3.3.4.tgz
gunzip Python-3.3.4.tgz && tar -xvf Python-3.3.4.tar
cd Python-3.3.4
./configure --prefix=/home/me/mypython && make && make install
cd /home/me/mypython
```

2. Get easy_install.
```
curl https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py | ./bin/python3.3
```

3. Get pip  
```
./bin/easy_install-3.3 pip
```

4. Get virtualenv  
```
./bin/pip3.3 install virtualenv
```

5. Create a virtualenv  
```
./bin/virtualenv ~/myprojectenv
cd ~/myprojectenv
```

6. Activate your virtualenv  
```
. bin/activate
```

7. Get Snakemake  
```
pip install snakemake
pip install rpy2
```

###Possible issues
Some servers may demand `--no-check-certificate` when running wget. You might need to actually alter `ez_setup.py` as below:

```
cmd = ['wget', url, '--quiet', '--no-check-certificate','--output-document', target]
```

### I don't have an R built as a shared library so rpy2 doesn't install

Build R from source, install locally:
```
wget http://cran.rstudio.com/src/base/R-3/R-3.0.2.tar.gz
gunzip R-3.0.2.tar.gz 
tar -xvf R-3.0.2.tar 
cd R-3.0.2
./configure --enable-R-shlib --prefix=/nas/is1/bin/R-3.0.2
make
make install
```

Edit your .bash_profile
```  
export R_HOME="/nas/is1/bin/R-3.0.2"
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$R_HOME/bin
```

Source it
```
source ~/.bash_profile
```

Make sure you are in the virtualenv
```
source /nas/is1/bin/variome-env/bin/activate
```

Install rpy2 from source
```
wget http://downloads.sourceforge.net/project/rpy/rpy2/2.2.x/rpy2-2.2.1.tar.gz
gunzip rpy2-2.2.1.tar.gz 
tar -xvf rpy2-2.2.1.tar 
python setup.py build --r-home /nas/is1/bin/R-3.0.2/ install
```