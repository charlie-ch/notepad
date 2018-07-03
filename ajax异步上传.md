

###  ajax异步上传
**1. html页面**
```
<input type="file" name="excelFile" id="planDetailExcelFile" style="display:none">
<a href="#" class="btn btn-primary" id="importPlanDetailBtn" data-toggle="modal"> 
     <i class=" md-file-download"></i> <span th:text="#{checkPlan.import}">导入</span>
</a>
```
**2. 点击导入**
```
$("#importPlanDetailBtn").click(function(){
	//触发file输入框的change事件
	 $("#addCheckPlanDiv #importCheckPlanDetailModal  #planDetailExcelFile").trigger("click");
})
```
**3. file 输入框change事件**
```
$("#addCheckPlanDiv #planDetailExcelFile").change(function(){
var formData = new FormData();
//其他表单值赋值方式
formData.append("sapdSourceid",checkPlan.sapId);
//文件
formData.append("excelFile", $("#addCheckPlanDiv #planDetailExcelFile")[0].files[0]);   
     $.ajax({
         url: basePath+"PlanDetail/import",
         type: "POST",//必须post
         data: formData,
         /**
         *必须false才会自动加上正确的Content-Type
         */
         contentType: false,
         /**
         * 必须false才会避开jQuery对 formdata 的默认处理
         * XMLHttpRequest会对 formdata 进行正确的处理
         */
         processData: false,
         success: function (res) {
           //提示成功或失败
          //清空文件
          $("#addCheckPlanDiv #planDetailExcelFile").val('')
         },
         error: function () {
         //提示网络错误
        	 //清空文件
             $("#addCheckPlanDiv #planDetailExcelFile").val('')
         }
     });  
}
     
```

**4.后台控制器**
```
package com.pc.activiti.business.sysmanage.controller;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.util.Date;
import java.util.List;

import javax.annotation.PostConstruct;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang3.StringUtils;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;

import com.czy.framework.mybatis.support.keygen.IdWorker;
import com.czy.framework.mybatis.support.mapper.ConditionGroup;
import com.czy.framework.mybatis.support.mapper.QueryParam;
import com.github.pagehelper.PageHelper;
import com.pc.activiti.business.base.BootstrapPage;
import com.pc.activiti.business.common.util.IOUtils;
import com.pc.activiti.business.common.util.SystemPropertiesUtil;
import com.pc.activiti.business.sysmanage.entity.SqmUploadFile;
import com.pc.activiti.business.sysmanage.entity.param.SqmUploadFileParam;
import com.pc.activiti.business.sysmanage.service.DeptManageService;
import com.pc.activiti.business.sysmanage.service.SqmSysUserService;
import com.pc.activiti.business.sysmanage.service.SqmUploadFileService;
import com.pc.activiti.core.system.entity.SecurityUser;
import com.pc.activiti.core.util.Res;
import com.pc.activiti.core.util.SecurityUtils;

@Controller
@RequestMapping("/sqmuploadfile")
public class SqmUploadFileController {
	private static final Logger LOGGER = Logger.getLogger(SqmUploadFileController.class);
	//文件大小限制 100M
	private static long MAXLENGTH = 100*1024*1024;
	@Autowired
	private SqmUploadFileService sqmUploadFileService;
	@Autowired
    private SqmSysUserService sqmSysUserService;
	@Autowired
    private DeptManageService deptManageService;
	
	private String uploadPath;
	
	
	@PostConstruct
    void init() {
		uploadPath = SystemPropertiesUtil.getProperty("sqm.upload.file.path");
		if(StringUtils.isBlank(uploadPath)){
			uploadPath = "/u02/smscs/attach/";
		}
        File pathFile = new File(uploadPath);
        if (!pathFile.exists() && !pathFile.mkdirs()) {
            throw new IllegalStateException("创建附件上传目录失败");
        }
    }
	/**
	 * 上传附件
	 * @param excelFile
	 * @param sapdSourceid 业务表主键
	 * @param sourceNumber 业务表编号
	 * @return
	 */
	@RequestMapping("upload")
	@ResponseBody
	public Res uploadFile(@RequestParam("uploadFile") MultipartFile file,
			String sourceId, String sourceNumber){
		if(StringUtils.isBlank(sourceId) || null==file){
			return Res.error();
		}
		if(MAXLENGTH < file.getSize()){
			return Res.error();
		}
        // 生成文件名
        String newFileName = uploadPath+""+IdWorker.get32UUID() + "#_" + file.getOriginalFilename();
        File saveFile = new File(newFileName);
        // 写入文件流
        FileOutputStream out;
		try {
			out = new FileOutputStream(saveFile);
			IOUtils.clone(file.getInputStream(), out);
		} catch (Exception e) {
			LOGGER.error(e);
			return Res.error();
		}

        //保存附件信息
        SecurityUser currentLoginUser = SecurityUtils.getUserInfo();
        SqmUploadFile uploadFile = new SqmUploadFile();
        uploadFile.setSourceid(sourceId);
        uploadFile.setSourcenumber(sourceNumber);
        uploadFile.setFilename(file.getOriginalFilename());
        uploadFile.setFilepath(newFileName);
        uploadFile.setCreateby(currentLoginUser.getId());
        uploadFile.setCreatedate(new Date());
        uploadFile.setDeptid(currentLoginUser.getDeptId());
        sqmUploadFileService.insert(uploadFile);
        
		return Res.ok();
	}
	/**
	 * 分页查询
	 * @param sqmUpload
	 * @return
	 */
	@RequestMapping("/queryList")
	@ResponseBody
	public BootstrapPage<SqmUploadFile> queryList(SqmUploadFileParam sqmUpload) {
		PageHelper.startPage(sqmUpload.getPageNum(), sqmUpload.getLimit());
		//查询参数封装
		ConditionGroup conditionGroup = ConditionGroup.init();
		conditionGroup.andEq("sourceid", sqmUpload.getSourceId());
		conditionGroup.andEq("sourcenumber", sqmUpload.getSourceNumber());
		QueryParam queryParam = QueryParam.init();
		queryParam.and(conditionGroup);
		queryParam.desc("id");//按照主键降序排序
		
		List<SqmUploadFile> list = sqmUploadFileService.findListByParam(queryParam);
		String userId = null;
		SecurityUser currentLoginUser = SecurityUtils.getUserInfo();
		for(SqmUploadFile file:list){
			if(null==file || StringUtils.isBlank(file.getCreateby())){
				continue;
			}
			//姓名
			userId = new String(file.getCreateby());
			file.setCreateby(sqmSysUserService.selectUserNameByIds(file.getCreateby()));
			file.setDeptName(deptManageService.selectDeptNameByIds(file.getDeptid()));
			//判断能否删除
			if(userId.equals(currentLoginUser.getId())){
				file.setStatus("1");
			}
		}
		BootstrapPage<SqmUploadFile> pageInfo = new BootstrapPage<SqmUploadFile>(list);
		
		return pageInfo;
	}
	/**
	 * 删除附件
	 * @param id
	 * @return
	 */
	@RequestMapping("/delete")
	@ResponseBody
	public Res delete(Long id) {
        if (id == null) {
            return Res.error("id 不能为空");
        }
        sqmUploadFileService.deleteByPK(id);
        return Res.ok();
    }
	/**
	 * 下载附件
	 * @param id
	 * @return
	 */
	@RequestMapping("/download")
	public void download(HttpServletRequest request, HttpServletResponse resp) {
		String id = request.getParameter("id");
        if(StringUtils.isBlank(id)) {
             return;
        }
        SqmUploadFile uploadfile = sqmUploadFileService.findByPK(Long.parseLong(id));
        if(null==uploadfile){
        	 return;
        }
        try{
	        //文件全路径
	        String filePathName = uploadfile.getFilepath(); 
	        String fileName = new String(uploadfile.getFilename().getBytes("UTF-8"), "ISO8859-1");
	        // 输入流
	        FileInputStream input = new FileInputStream(filePathName);
	        resp.setContentType("application/force-download");
	        resp.setHeader("Content-Disposition", "attachment;filename="+fileName);
	        // 获取绑定了客户端的流
	        ServletOutputStream output = resp.getOutputStream();
	
	        // 把输入流中的数据写入到输出流中
	        IOUtils.clone(input,output);
	        input.close();
        }catch(Throwable e){
        	LOGGER.error(e);
        }
    }
}

```
**5.数据库**
```
CREATE TABLE `sqm_upload_file` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `filename` varchar(1000) DEFAULT NULL COMMENT '文件名',
  `filepath` varchar(1000) DEFAULT NULL COMMENT '文件路径',
  `sourceid` varchar(50) DEFAULT NULL COMMENT '业务表id',
  `sourcenumber` varchar(50) DEFAULT NULL COMMENT '业务表编号',
  `deptid` varchar(100) DEFAULT NULL COMMENT '创建部门',
  `createby` varchar(200) DEFAULT NULL COMMENT '流程节点名称',
  `createdate` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `status` varchar(10) DEFAULT '0' COMMENT '状态 0；有效，1：已删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `UN_SQM_UPLOAD_FILE` (`id`),
  KEY `IDX_SQM_UPLOAD_FILe` (`sourceid`,`sourcenumber`)
) ENGINE=InnoDB AUTO_INCREMENT=134 DEFAULT CHARSET=utf8 COMMENT='上传的附件信息';
```
    
