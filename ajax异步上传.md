

###  ajax异步上传
**java**
```
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
             if (res.code==200) {
            	 swal({
         			title : addPlanDetailPageMessage.successMsg,
         			type : "success",
         			showConfirmButton : true,
          			confirmButtonText : addPlanDetailPageMessage.confirm,// 确认按钮文字
         		});
            	 $("#addCheckPlanDiv #planDetailExcelFile").val("");
            	 $('#addCheckPlanDiv #checkDetailTable').bootstrapTable('refresh');
            	//刷新检查计划页面
 				if(checkPlan.sapType=="0"){
 					$('#checkPlanPage #checkPlanTable').bootstrapTable('refresh');
 				}else{
 					$('#specialCheckPlanPage #checkPlanTable').bootstrapTable('refresh');
 				}
             }else {
            	 /*swal({
          			title : addPlanDetailPageMessage.errorMsg,
          			type : "error",
          			text:"123<span style='color:red'>"+res.msg+"</span>",
          			showConfirmButton : true,
          			confirmButtonText : addPlanDetailPageMessage.confirm,// 确认按钮文字
          		});*/
            	 
            	 $("#addCheckPlanDiv #errrorMsg").val(res.msg);
            	 $("#addCheckPlanDiv #importErrorMsgModal").modal('show');
             }
          //清空文件
          $("#addCheckPlanDiv #planDetailExcelFile").val('')
         },
         error: function () {
        	 swal({
       			title : addPlanDetailPageMessage.netErrorMsg,
       			type : "error",
       			text:addPlanDetailPageMessage.pleaseRefresh,
       			showConfirmButton : true,
       			confirmButtonText : addPlanDetailPageMessage.confirm,// 确认按钮文字
       		});
        	 //清空文件
             $("#addCheckPlanDiv #planDetailExcelFile").val('')
         }
     });     
```
    
