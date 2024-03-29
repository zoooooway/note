# API 设计
## 共通原则
* 使用全局版本：格式为 /api/{版本}/user/get，考虑服务做大版本升级的时候进行切换；
* 使用URL的最后一部分代表操作类型 get/post/delete/create，`HTTP`不限制`GET`或者`POST`方式；
* 所有资源的`URL`必须由服务端进行拼接，而不是客户端。理由是当`URL`发生变更的时候，不需要考虑客户端的兼容性。
  * 比如预览视频的地址，如果服务的不进行拼接而交由客户端进行拼接，那么后续变更`URL`后客户端就无法解析了。
* 默认采用GZIP压缩策略，一般超过2K的数据压缩比就比较高了；

## 资源视角的API设计原则
* 通过领域、资源方式设计`API`，即资源需要成为`URL`的一部分，`ID`需要作为`Path`的一部分；
* 对于数据较多的内容，需要使用过滤器，即 `Partial Select`，在一些情况下，服务端可以根据过滤条件进行优化；
* 分页获取的时候统一采用 /?limit=xxx&offset 的方式
* 资源的`GET`操作尽可能不交叉，即每个`API`尽可能只返回该资源下的数据，而不关心其他资源的数据；
  > 如设备列表需要显示该设备是否有分享过，而设备和分享属于两个“资源”
* 如果出于效率的需要，希望做`API`合并，则有两种做法：
  * 不合并，客户端加载的时候异步加载其他需要的数据；
  * 需要1）加入`Filter`（表明是否需要交叉的资源）； 2）需要针对`API`不返回交叉数据时的处理方法；