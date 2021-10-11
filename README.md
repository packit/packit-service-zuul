# packit-service-zuul

## About Zuul CI - softwarefactory-project.io

[Zuul](https://zuul-ci.org) is a CI system invented in the OpenStack community.
[Softwarefactory](https://softwarefactory-project.io) (SF) is project which is hosting services (Zuul being one of them) to ease development of open source projects.

## How it works in our case

We have [softwarefactory-project-zuul](https://github.com/apps/softwarefactory-project-zuul) GitHub App [installed to all packit-service repos](https://github.com/organizations/packit-service/settings/installations).
We have a separate [packit-service tenant in Zuul](https://softwarefactory-project.io/zuul/t/packit-service/status).
It uses the configuration from this repository, see especially [jobs.yaml](https://github.com/packit-service/packit-service-zuul/blob/main/zuul.d/jobs.yaml) and [pipelines.yaml](https://github.com/packit-service/packit-service-zuul/blob/main/zuul.d/pipelines.yaml) in [zuul.d/](https://github.com/packit-service/packit-service-zuul/tree/main/zuul.d).
We can easily use those pipelines & jobs from other repositories so if you have a job which could be used by more repositories, you can add it to [jobs.yaml](https://github.com/packit-service/packit-service-zuul/blob/main/zuul.d/jobs.yaml).

If you want to enable Zuul in another packit-service repository, just create a `.zuul.yaml` and define `noop` job inside to test the connection, see [example](https://github.com/packit-service/packit-service-zuul/pull/8/files).
Once the testing is done, there is a new comment posted in the PR with links to each job.
If it looks OK, add any other jobs you want and remove the `noop` one.

Your locally cloned repo is: `project_dir: "{{ ansible_user_dir }}/{{ zuul.project.src_dir }}"`

## More useful links

- [Zuul Documentation](https://zuul-ci.org/docs), esp. [Job](https://zuul-ci.org/docs/zuul/reference/job_def.html) & [Job Content](https://zuul-ci.org/docs/zuul/reference/jobs.html) in [User Reference](https://zuul-ci.org/docs/zuul/reference/user.html)
- [zuul/zuul-jobs](https://opendev.org/zuul/zuul-jobs) - Ansible job definitions for Zuul. [Roles](https://opendev.org/zuul/zuul-jobs/src/branch/master/roles) are [used](https://zuul-ci.org/docs/zuul/reference/job_def.html#attr-job.roles) by the [base job](https://github.com/packit-service/packit-service-zuul/blob/main/zuul.d/jobs.yaml#L10).
- [Our tenant config](https://softwarefactory-project.io/cgit/config/tree/resources/tenant-packit-service.yaml) - To submit a change (like [this](https://softwarefactory-project.io/r/18820)) follow [The Config repository](https://softwarefactory-project.io/docs/user/config_repo.html) and [Contribute to a project](https://softwarefactory-project.io/docs/user/index.html) docs.
- [OpenStack's repo](https://opendev.org/zuul/zuul-jobs/src/branch/master) with existing jobs, playbooks and roles and [repositories which use it](https://opendev.org/openstack).
- [Available nodes](https://softwarefactory-project.io/cgit/config/tree/nodepool/diskimages/package.dhall).
- Questions: Internal IRC channel #rhos-ops or contact directly [Fabien Boucher](https://github.com/morucci).
