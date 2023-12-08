---
title: note
comments: true
aside: true
top_img: false
date: 2022-01-28 13:36:24
tags:
description: note
mathjax:
katex:
categories:
top: true
cover: http://42.192.149.99:9999/down/YK6LeoksZfw5?fname=/360wallpaper.jpg
---
### 数据库返回数据操作
> 查询list不能删除list中元素，因为再次查询会为空值。原因：可能存在缓存。

* 如下代码：

```java
/**删除返回值list元素操作
*/
GenerateReportOperateDTO operateDTO = new GenerateReportOperateDTO();
operateDTO.setUseType(ReportUseType.SERVICE_PROVIDER.getCode());
operateDTO.setProdId(product.getProdId());
List<GenerateReportOperateDTO> dbGenerateReportOperateDTOList =
     Optional.ofNullable(generateReportOperateRepository.queryList(operateDTO)).orElse(new ArrayList<>());

List<GenerateReportOperateDTO> dbList = Lists.newArrayList(dbGenerateReportOperateDTOList);
int cnt = 0;
for (ProductReportManage productReportManage : productReportManageList) {
    String[] reportTypes = productReportManage.getReportType().split("\\|");
    for (String reportType : reportTypes) {
        GenerateReportOperateDTO in = new GenerateReportOperateDTO();
        in.setProdId(product.getProdId());
        in.setPosSs(posSsEnum.getCode());
        in.setReportType(reportType);
        in.setTransType(productReportManage.getTransType());
        in.setUseType(ReportUseType.SERVICE_PROVIDER.getCode());

        boolean removed = dbList.removeIf(db -> db.getProdId().equals(in.getProdId())
                && db.getPosSs().equals(in.getPosSs())
                && db.getReportType().equals(in.getReportType())
                && db.getTransType().equals(in.getTransType())
                && db.getUseType().equals(in.getUseType()));
        if (!removed){
            logger.info("自动生成内控报表, [运营报表生成配置表第一次初始化]，产品号：{}，报表分类：{}，报表类型：{}，交易类型：{}，使用类型：{}",
                    in.getProdId(),
                    in.getPosSs(),
                    in.getReportType(),
                    in.getTransType(),
                    in.getUseType());
            generateReportOperateRepository.create(in);
            cnt++;
        }
    }
}
/**再次查询
*/
GenerateReportOperateDTO operateDTO = new GenerateReportOperateDTO();
operateDTO.setUseType(ReportUseType.SERVICE_PROVIDER.getCode());
operateDTO.setProdId(reqProdId);
List<GenerateReportOperateDTO> generateReportOperateDTOList = generateReportOperateRepository.queryList(operateDTO);
if (CollectionUtils.isEmpty(generateReportOperateDTOList)) {
	// 会为空
    logger.info("Bh9999自动生成内控报表完成，数据为空，跑批日期{},产品号：{}", bhdate, reqProdId);
    return;
}
```

# Have fun ^_^
---