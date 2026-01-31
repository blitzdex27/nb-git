# Bundling commit

Bundle

```
git bundle create <your-custom-file-name>.bundle HEAD~10..HEAD
```

Note: The `HEAD~10..HEAD` is the rand og commit. Also, the lower limit should include at least one commit that also exists on the receiver, since they are the dependencies for the commits inside this bundle. The more the better.

Unbundle

```
git pull <path-of-file>
```