# VolumeClaimTemplates：专属的持久化“储物柜”

在 Deployment 中，如果你定义了一个 Volume，所有副本（Pod）都会共享这同一个 Volume。这对 Web 服务没问题，但对数据库来说是灾难——你不能让三个 MySQL 实例往同一个文件夹里乱写数据。

# 作用

VolumeClaimTemplates 是一个 **PVC（PersistentVolumeClaim）的模板**。

1. **自动创建**：每当 StatefulSet 创建一个 Pod（如 `web-0`），它会根据模板自动创建一个对应的 PVC（如 `data-web-0`）。
2. **一对一绑定**：Pod `web-0` 只能挂载 `data-web-0`。
3. **状态保留**：如果 `web-0` 删除了，对应的 PVC **不会被自动删除**。当 `web-0` 被重新创建时，它会精准地找回属于它的 `data-web-0`。