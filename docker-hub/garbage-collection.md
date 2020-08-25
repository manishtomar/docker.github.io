---
description: Garbage collection
keywords: Docker, image deletion, garbage
title: Garbage collection
---

Docker Hub has automated garbage collection system setup that will delete
unreferences images and associcated blobs/layers after a certain period of time.

An unreferenced image is one that is not referred by any tag in any repository.
For example, if user pushes image with `digest1` and tag it as `user1/repo1:tag1` then
that `digest1` image will be referenced and not deleted. When user deletes the tag `tag1` from
Hub UI then that image `digest1` is not referenced by any tag and is a candidate for
deletion. If for example this same image was tagged by another tag (say `user1/repo2:tagn`)
then that image is still referenced and will not be deleted.

If the user pushes another image over an existing tag (say `digest2` on `user1/repo1:tag1`)
then `digest1` although not accessible via tag is still considered referred and is not deleted.
This is done party because orchestrators like Kubernetes and Swarm always pull images by digest
and refer to the "old" image independent of updated tag. Depending on orchestrator setting it
may use old version (`digest1`) or new version (`digest2`) of the tag. Hub will not know whether
to delete the old image or not. Hence, it keeps it around. However, if `tag1` is deleted
then both `digest1` and `digest2` are candidates for deletion (unless some other tags refer to them).

The garbage collection system internally first hides the image and associated blobs for some period
of time before completely purging it. This allows us to recover an image/blob if such a need arises.

## Relation to Inactive Image retention policy

This garbage collection system has been running for more than 1 year and is unrelated to [Inactive
Image retention policy](https://www.docker.com/pricing/resource-consumption-updates).
While the same system may be used internally to delete the image,
the decision to find inactive image is defined in that policy.
