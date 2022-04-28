# packit-service-zuul

## About Zuul CI - softwarefactory-project.io

[Zuul](https://zuul-ci.org) is a CI system invented in the OpenStack community.
[Softwarefactory](https://softwarefactory-project.io) (SF) is project which is hosting services (Zuul being one of them) to ease development of open source projects.

## How it works in our case

We have [softwarefactory-project-zuul](https://github.com/apps/softwarefactory-project-zuul)
GitHub App [installed to all packit-service repos](https://github.com/organizations/packit-service/settings/installations).
We have a separate [packit-service tenant in Zuul](https://softwarefactory-project.io/zuul/t/packit-service/status).
It uses the configuration from this repository, see especially
[jobs.yaml](https://github.com/packit-service/packit-service-zuul/blob/main/zuul.d/jobs.yaml)
and [pipelines.yaml](https://github.com/packit-service/packit-service-zuul/blob/main/zuul.d/pipelines.yaml)
in [zuul.d/](https://github.com/packit-service/packit-service-zuul/tree/main/zuul.d).
We can easily use those pipelines & jobs from other repositories so if you have
a job which could be used by more repositories, you can add it to
[jobs.yaml](https://github.com/packit-service/packit-service-zuul/blob/main/zuul.d/jobs.yaml).

To enable Zuul in another repository, add it to
[our tenant config](https://softwarefactory-project.io/cgit/config/tree/resources/tenant-packit-service.yaml)
by following [The Config repository](https://softwarefactory-project.io/docs/user/config_repo.html)
and [Contribute to a project](https://softwarefactory-project.io/docs/user/index.html), specifically:

- Add your ssh key to https://softwarefactory-project.io/r/settings/#SSHKeys if it's not already there.
  (Last time, my [3072b RSA key didn't work](https://dev.to/bowmanjd/upgrade-ssh-client-keys-and-remote-servers-after-fedora-33-s-new-crypto-policy-47ag),
  so I had to generate a ED25519 key.)
- `git clone https://softwarefactory-project.io/r/config`
- Edit `resources/tenant-packit-service.yaml` and commit
- `dnf install git-review` & `git review`
  (Last time I had to do `git review -R`, install suggested hook and amend)
- Follow the review ticket and once it's approved and merged the configuration will be updated.

Then create a `.zuul.yaml` and define `noop` job inside to test the connection,
see [example](https://github.com/packit-service/packit-service-zuul/pull/8/files).
Once the testing is done, there is a new comment posted in the PR with links to each job.
If it looks OK, add any other jobs you want and remove the `noop` one.

Your locally cloned repo is: `project_dir: "{{ ansible_user_dir }}/{{ zuul.project.src_dir }}"`

## More useful links

- [Zuul Documentation](https://zuul-ci.org/docs), esp. [Job](https://zuul-ci.org/docs/zuul/reference/job_def.html) & [Job Content](https://zuul-ci.org/docs/zuul/reference/jobs.html) in [User Reference](https://zuul-ci.org/docs/zuul/reference/user.html)
- [zuul/zuul-jobs](https://opendev.org/zuul/zuul-jobs) - Ansible job definitions for Zuul. [Roles](https://opendev.org/zuul/zuul-jobs/src/branch/master/roles) are [used](https://zuul-ci.org/docs/zuul/reference/job_def.html#attr-job.roles) by the [base job](https://github.com/packit-service/packit-service-zuul/blob/main/zuul.d/jobs.yaml#L10).
- [OpenStack's repo](https://opendev.org/zuul/zuul-jobs/src/branch/master) with existing jobs, playbooks and roles and [repositories which use it](https://opendev.org/openstack).
- [Available nodes](https://softwarefactory-project.io/cgit/config/tree/nodepool/diskimages/package.dhall).
- Zuul/Gerrit questions & issues:
  - Internal IRC channel #rhos-ops
  - softwarefactory-operations-team@redhat.com

## Debug locally in a Zuul VM

To debug issues related with the _Zuul VM_ you can follow this steps:

- Install some _virt tools_ and Ansible:

```
$ sudo dnf install libvirt /usr/bin/virt-customize virt-manager ansible
```

- Clone `https://softwarefactory-project.io/r/config` and `cd` into it.

- Add ansible, if you need to run a playbook inside the VM, to the list of installed packages

```patch
diff --git a/nodepool/virt_images/cloud-fedora-35.yaml b/nodepool/virt_images/cloud-fedora-35.yaml
index b00d4b2e..385b955c 100644
--- a/nodepool/virt_images/cloud-fedora-35.yaml
+++ b/nodepool/virt_images/cloud-fedora-35.yaml
@@ -43,6 +43,7 @@
       - python3-pyyaml
       # For add-build-sshkey
       - python3-libselinux
+      - ansible
   tasks:
     - block:
         - import_role:
```

- Provide your public key to be able to SSH into the VM later, following this patch example

```patch
diff --git a/nodepool/virt_images/roles/zuul-user/tasks/main.yaml b/nodepool/virt_images/roles/zuul-user/tasks/main.yaml
index e771c27d..9eb8cd3e 100644
--- a/nodepool/virt_images/roles/zuul-user/tasks/main.yaml
+++ b/nodepool/virt_images/roles/zuul-user/tasks/main.yaml
@@ -8,7 +8,7 @@

 - name: Prepare the authorized_keys file
   copy:
-    src: /var/lib/nodepool/.ssh/zuul_rsa.pub
+    src: /home/maja/.ssh/id_rsa.pub
     dest: "{{ image_tmp_dir }}/authorized_keys"
     remote_src: true
```

- Build the _Zuul VM qcow2 image_; changing the `image_output` variable to a suitable location.

```
$ ansible-playbook -vvv -c local -i localhost, -e qcow2_type=true -e image_output=/home/maja/softwarefactory.io/zuul -bK nodepool/virt_images/cloud-fedora-35.yaml
```

- Use the _Virtual Machine Manager_ to create a new VM:

  - File -> New Virtual Machine
  - Choose Import existing disk image
  - Browse -> Browse Local to the `image_output` dir and select _zuul.qcow2_
  - Type _Fedora XX_ as your operating system type
  - Create the new VM with the other wizard default settings

- ssh into the VM like in the example; once the VM is started its **IP** is _displayed above the login prompt_

```
$ ssh zuul-worker@192.168.122.253
```

- If you need to debug the hardly CI, as an example, copy the hardly project into the VM and enter the VM again:

```
$ scp -r /home/maja/PycharmProjects/hardly/ zuul-worker@192.168.122.253:/home/zuul-worker
$ ssh zuul-worker@192.168.122.253
```

- Start the ansible playbook for the CI with something like the following

```
$ cd hardly
$ ansible-playbook -vvv -c local -i localhost, -e '{"zuul": {"branch": "main", "project": {"src_dir": "/home/zuul-worker/hardly"}}}' files/zuul-tests.yaml
```
