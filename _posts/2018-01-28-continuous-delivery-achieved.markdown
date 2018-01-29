---
layout: post
title:  "Continuous Delivery Achieved!"
date:   2018-01-28
categories: update
---
I spent a couple hours today putting together a nice solution for
"continuous delivery" of this website. I had a few requirements for
this:
1. Deploy based on a git commit to the master branch.
2. Deploy the build on Travis CI after integration testing.
3. Use ssh to deploy to my server
4. Do not publicize the hostname of my server or my ssh keys.

I happened across this [lovely tutorial] to do almost everything I
intended, but I had to change a few things. The key steps are:
1. Generate an ssh key pair.
2. Use travis cli to encrypt the public key.
3. Place the private key on the website's host server.

    ```
    ssh-keygen -t rsa -b 4096 -C 'build@travis-ci.org' -f ./deploy_rsa
    travis encrypt-file deploy_rsa --add
    ssh-copy-id -i deploy_rsa.pub <ssh-user>@<deploy-host>

    rm -f deploy_rsa deploy_rsa.pub
    git add deploy_rsa.enc .travis.yml
    ```

4. Set environment variables in travis for the host name and
    website path.
5. Update [.travis.yml] to decrypt and import the ssh key.

    ```
    before_deploy:
    - openssl aes-256-cbc -K $encrypted_77244c28eaee_key -iv $encrypted_77244c28eaee_iv
      -in deploy_rsa.enc -out /tmp/deploy_rsa -d
    - eval "$(ssh-agent -s)"
    - chmod 600 /tmp/deploy_rsa
    - ssh-add /tmp/deploy_rsa
    - ssh-keyscan -t 'rsa,dsa,ecdsa' -H $deploy_host_name 2>&1 | tee -a $HOME/.ssh/known_hosts
    ```

6. Update [.travis.yml] to deploy the build to the host server.

    ```
    deploy:
      provider: script
      skip_cleanup: true
      script: rsync -r --delete-after --quiet $TRAVIS_BUILD_DIR/_site/* $deploy_host
      on:
    branch: master
    ```

When I followed the tutorial, I deviated in that I did not want to
reveal the location of my deploy host or path. However, when I tried to
use the built-in Travis CI addon to set the ssh-known-hosts, I found
that it didn't work with the environment variables in Travis CI. I
found a [ticket on Travis CI's Github] that had a possible workaround,
so I tried it and got it working as you can see in the snippet above.

The deploy is still not perfect as right now it is just a copy operation
and will not handle deleting files, for instance. Later I can add some
more operations to do a cleaner deploy, but as I'm mostly *adding* things
to the site now, this setup will suffice for a while.

[lovely tutorial]: https://oncletom.io/2016/travis-ssh-deploy/
[.travis.yml]: https://github.com/andrewfister/personal-website/blob/master/.travis.yml
[ticket on Travis CI's Github]: https://github.com/travis-ci/travis-ci/issues/7043#issuecomment-329308184