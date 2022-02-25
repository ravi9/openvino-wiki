The OpenVINO repository has several branches with different contribution policies.

## Branches and contribution policies in OpenVINO

Each OpenVINO release has its own release branch names as `releases/<year>/<release number>[.<patch number>]`:

* [releases/2021/4](https://github.com/openvinotoolkit/openvino/tree/releases/2021/4)
    * **LTS** (long term support) branch
    * **only** critical fixes can be sent
* [releases/2021/1](https://github.com/openvinotoolkit/openvino/tree/releases/2021/1)
    * no patches are accepted
* [releases/2021/2](https://github.com/openvinotoolkit/openvino/tree/releases/2021/2)
    * current release branch.
* [releases/2021/3](https://github.com/openvinotoolkit/openvino/tree/releases/2021/3)
    * no patches are accepted
* [master](https://github.com/openvinotoolkit/openvino/tree/master)
    * development branch, can accept both fixes and new functionality:
       - bug fixes
       - optimizations
       - documentation improvements
       - sample improvements or adding new samples for existing functionality
       - small improvements or new features which don't break compatibility with previous releases
    * **both** fixes and new functionality can be sent to this branch.
    * source compatibility must be preserved. Source incompatible changes can be merged only before `releases/2022/1` release.

> **NOTE**: All other branches are **read-only**.

## Cherry-picking changes from `release` branch to `master`

If you have fixed a bug in OpenVINO release branch using:

```sh
$ git checkout releases/2020/3
$ git checkout -b bug-fix
$ git add <files>
$ git commit --message="Fixing issue #XXX"
$ git push origin big-fix
```

you need to manually cherry-pick commits with fixes to master branch as well:

```sh
$ git checkout master
$ git checkout -b bug-fix-port-master
$ git cherry-pick bug-fix # resolve conflicts after if any
$ git push origin bug-fix-port-master
```

## Related articles

* [GitHub Flow](https://guides.github.com/introduction/flow/) guide
* [Forking Projects](https://guides.github.com/activities/forking/) guide