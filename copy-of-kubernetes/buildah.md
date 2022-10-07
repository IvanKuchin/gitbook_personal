# Buildah

## ISSUE: Error during unshare(CLONE\_NEWUSER): Operation not permitted

1. Try to run it using podman
2. Try to add –device /dev/fuse
3. Try to add –privileged to container run  &#x20;

```
(non-root user)$ docker run --privileged --rm -it quay.io/buildah/stable bash
```

This command can be run from non-priviledged user using `--priviledged` keyword

Check this link for more details: [https://github.com/containers/buildah/issues/1469](https://github.com/containers/buildah/issues/1469)

