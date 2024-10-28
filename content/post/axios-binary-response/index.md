+++
author = "H. Wang"
title = "axios请求二进制流或json处理"
date = "2024-06-06"
description = "axios请求结果判断，对二进制流或json做不同处理"
tags = [
    "前端", "axios", "JS"
]
categories = [
    "JS"
]

image = ""
+++

对于特殊的上传需求：上传成功后展示成功相关信息的提示语，而失败通过Excel文件下载展示详细的错误数据及错误原因。通常可以在出错后生成Excel文件，并保存至服务器或上传到minio，再返回文件地址，这样返回结果类型统一。

也可在错误时直接通过response返回二进制数据流，需要在前端请求后判断响应的数据类型。

```javascript
axios.post(
  this.upload.url,
  params,
  {
    responseType: 'blob',
    headers: this.upload.headers
  }
).then(response => {
  if (response.data.type === 'application/json') {
    return response.data.text()
  } else {
    this.$confirm('批量导入信息失败，请下载失败文件查看详情！', '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    }).then(() => {
      const blob = new Blob([response.data])
      const a = document.createElement('a')
      const url = URL.createObjectURL(blob)
      const filename = '批量导入失败原因文件' + '.xlsx'
      a.href = url
      a.download = filename
      a.click()
      URL.revokeObjectURL(url)
      this.upload.open = false
      this.upload.isUploading = false
      this.$refs.upload.clearFiles()
    })
  }
}).then(text => {
  if (!text) return
  const response = JSON.parse(text)
  this.upload.open = false
  this.upload.isUploading = false
  this.$refs.upload.clearFiles()
  let title = ''
  if (response.code === 200) {
    title = response.msg
  } else {
    title = response.message
  }
  this.$alert(
    '<div style=\'overflow: auto;overflow-x: hidden;max-height: 70vh;padding: 10px 20px 0;\'>' + title + '</div>',
    '导入结果',
    { dangerouslyUseHTMLString: true }
  )
  this.getDeptTree()
}).finally(() => {
    this.loading = false
})
```