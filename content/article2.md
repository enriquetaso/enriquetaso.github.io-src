Title: Working in my first RBD bug 
Date: 2016-08-19 10:02

# Working in my first RBD bug

The cinder manage command it’s only for independent rbd volumes. However,
manage an already-managed volumes is allow by the api and there’re not 
exception raised.

When you try to manage an already-managed volume, you received an unhandled
error: `UnboundLocalError: local variable ‘rbd_image’ referenced before assignment`

How to reproduce:

1. Create a volume:
```
cinder create 1 –name vol1
```
2. try to manage the volume (you can check the host with `cinder show <volume ID>`):
```
cinder manage <volume host> <volume ID>
```
3) On the c-vol you can appreciate the unhandled error.

When this happens the RBD driver **should handle the error**. The RBD driver should 
catch the exception and show a custom message notifying the user.

So, trying to handle the problem inside the rbd driver.
 

Coding the unit test is still a challenge for me. Thanks to my mentor I learned about
them and we added one to patch.

After coded it, It’s time to run the
[unit test](https://review.openstack.org/#/c/354289/1/cinder/tests/unit/volume/drivers/test_rbd.py)
in devstack:

1. Use an environment. One way to do it:
```
vagrant/cinder$ ./run_tests.sh -V
```
the `-V` will create a virtual env for you

2. Active the env:
```
vagrant/cinder$ source env/bin/activate
```
3. Check it, this should list all of the rbd tests, in that list should be our new one:
```
vagrant/cinder$ testr list-tests | grep RBDTestCase
```
 
4. Let’s run just that one test:
```
vagrant/cinder$ testr run cinder.tests.unit.volume.drivers.test_rbd.RBDTestCase.test_manage_existing_with_invalid_rbd_image
```
 
[Ver el articulo 1]({filename}category/article1.rst)
