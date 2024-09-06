- `git log branch1..branch2`: this command will show you all the commits that are reachable from `branch2` but not from `branch1`, effectively showing you the commits unique to `branch2` since it diverged from `branch1`.
  - `git log -p branch1..branch2`: also displays the diff of each commit; `-p` is for `--patch`
  
- [x]  What files were changed between two commits?
  - `git diff <commit-1> <commit-2> --stat`
  - Example::
	```shell
	➜  BrimstoneLinux git:(emr-backport-v1) g diff ebee89e0ad78~ ebee89e0ad78 --stat
	drivers/iommu/intel/iommu.c | 88 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++------
	drivers/iommu/intel/svm.c   |  5 ++++-
	include/linux/intel-iommu.h |  4 ++++
	3 files changed, 90 insertions(+), 7 deletions(-)
	```


TODO:
------------------------------------------------
