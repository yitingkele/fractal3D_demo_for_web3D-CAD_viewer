> 不同于obj/gltf/fbx等通用格式，为了保护自己的技术专利，主流的CAD软件公司在格式数据结构定义上通常采用闭源的方式，使得在网页上展示它们变得非常困难。 fractal 3D 可以支持在网页端流畅渲染几乎所有主流的 CAD 软件格式（参考[格式支持清单](https://ever-xyz.feishu.cn/wiki/wikcn04EgZRPnFYCFbhHHXYVDQb)）,使用的方法就是在将其转换成分形三维自主研发的e3dx格式。 通过本章的内容，你将掌握上传、转换并在网页显示这些复杂格式文件的方法。

## 文件转换的基本流程

> 你既可以选择直接通过http接口调用我们部署在云端的Saas转换服务，也可以选择在公司和组织的私有云/服务器集群中部署我们的文件转换服务。 如果你希望进一步了解私有化部署的具体方案和报价，请[联系我们](https://ever-xyz.feishu.cn/wiki/wikcnf3cKXdQGqiIiC3Ld7NS5Ud#LSYEdAoSCoUuoKxkpVdcIfEoneg)～

这里以SaaS模式为例，介绍一下文件转换的基本流程：

-   先将文件包装成[FormData](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)的形式发送至我们提供的上传接口，我们会将该文件对应的uuid通过返回值的形式告知你。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b928ad79457c44a5a3453cc2b5b9d8c7~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1280&h=883&s=103876&e=png&b=ffffff)



```
const formData = new FormData();
formData.append("file", file);
```

-   在转换过程中，你可以通过上一步返回的uuid 向分形三维的后台轮询该文件的转换状态，可能的状态包括：
-   pending（挂起）
-   succeed（成功）
-   failed（失败）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a621cc88bd74f7a927dd5745a018753~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1332&h=759&s=45846&e=png&b=ffffff)



```
// 上传文件到服务器
// 请注意这里是ts方便你了解类型， 如果直接放到html中需要去掉变量类型
// 这里的 FormData 以及 File 类型就是通用的 js 文件操作类型， 可在此处 https://developer.mozilla.org/en-US/docs/Web/API/File 以及 https://developer.mozilla.org/en-US/docs/Web/API/FormData 获取详情
async function uploadFile(file: File, onProgress: (progress: number) => void) {
    const formData = new FormData();
    formData.append('file', file);
    try {
        // 调用 uploadFile 方法，将包装好的 formData 上传到服务器
        const { data } = await axios.post('https://api.evercraft.co/3d/v1/file/upload', formData, {
            onUploadProgress(e) {
                const { loaded, total } = e;
                if (total) {
                    onProgress((loaded / total) * 100);
                }
            }
        });

        const { filename, uuid } = data.data;
        console.log(filename, uuid);
    } catch (error) {
        console.error(error);
    }
}
```

  


-   若转换文件成功，轮询的返回值中将包含e3dx文件的url。你既可以选择将其下载至本地的存储空间自行管理，也可以在下一次需要显示该文件时再次通过uuid向我们请求最新的url。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a6824536e894b4dbf72f7fb967ee00a~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1440&h=1170&s=253644&e=png&b=ffffff)



```
// 从服务器查询文件转换状态
async function pollingFileStatus() {
    // 使用 uploadFile 返回的 uuid 查询该文件的转换状态
    const { data } = await axios.get('https://api.evercraft.co/3d/v1/file', {
        params: { key: this._uuid }
    });

    const { status, url } = data;
    console.log(data);

    if (status === 'succeed') { // 转换成功
        url = `https://evercraft.co${url}`; // 拼接文件的 url
        status = 'succeed';
    } else if (status === 'pending' || status === 'running') { // 转换中
        status = 'pending';
    } else { // 转换失败
        status = 'failed';
    }

}
```

> 注意：每个e3dx文件的URL会在15分钟后失效，超过这个时间就必须通过uuid向分形三维后台请求新的url！

* * *

完整示例 请参考demos中的[upload-and-show.html](https://ever-xyz.feishu.cn/wiki/wikcnyNyTjCVgm6gyWoCCCDhuDb)