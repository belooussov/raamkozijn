# Raamkozijn
Running Windows inside a Docker container for test automation purposes. VirtualBox/qemu, vagrant, packer, ansible et al.

## Creating the Windows base image with Packer
A Windows 7 Enterprise SP1 x86 ISO needs to be used and needs to be available at ./packer-windows/iso/. For example "en_windows_7_enterprise_with_sp1_x86_dvd_u_677710.iso".

```
$ make packer-build-vbox (VirtualBox builder. Outputs a Vagrant box)
$ make packer-build-vbox-vtx (accelerated with VT-x)
$ make packer-build-qemu (Qemu builder)
```

The contents of the image can be modified using the Autounattend.xml file at ./packer-windows/answer_files/7/Autounattend.xml.
The resulting Vagrant box can in turn be found at ./packer-windows/windows_7_virtualbox.box after Packer has finished.

**Windows Updates are disabled by default**

### Setting a corporate proxy
The Windows VM might need to communicate with a proxy to download files. We use the cntlm package to do this. You can set the proxy uid/pw within the Autounattend.xml file at ./packer-windows/answer_files/7/Autounattend.xml.

```
> set_proxy.bat <host ip> <port>
```

```xml
<!-- Configure proxy settings -->
<SynchronousCommand wcm:action="add">
    <CommandLine>cmd.exe /c a:\set_proxy.bat 10.0.2.2 3128</CommandLine>
    <Description>Configure proxy</Description>
    <Order>1</Order>
    <RequiresUserInput>true</RequiresUserInput>
</SynchronousCommand>
```

## Using Vagrant and Ansible to provision the base OS (Dev environment)

```
$ vagrant up --no-provision
$ ansible-playbook provision.yml -e SeleniumGridHubFQDN=<grid hub> [-e SeleniumGridHubPort=80] -e proxyHost=<ip-address of host-only adapter> -e proxyPort=<probably 3128>  -e host=<ip-address of the host for the return call>
```

## Checking the WinRM connection with Ansible
```
$ ansible default -i ansible.ini -m win_ping -vvv
```
