### Disable KVM kernal extension
```bash
sudo rmmod kvm_amd   # or kvm_intel if Intel CPU
sudo rmmod kvm
```

Now,
```bash
vagrant up
vagrant ssh 
```


