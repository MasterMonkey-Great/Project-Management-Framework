# 2.5 Cocoapods常见问题以及解决方案



## 问题一 明明已经引入Pod库，头文件总是找不到


* 在TARGETS -> Search Paths -> User Header Search Paths 中 写入 ${SRCROOT}， 再将后面参数改为recursive

* 选择target -> BuildSettings -> search Paths 下的 User Header Search Paths, 添加 $(PODS_ROOT), 并将后面设置为recursive

* 选择target -> BuildSettings -> search Paths 下的 User Header Search Paths, 添加 $(BUILT_PRODUCTS_DIR), 并将后面设置为recursive