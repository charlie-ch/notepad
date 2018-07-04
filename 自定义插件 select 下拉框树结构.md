# 自定义插件 select 下拉框树结构
**js**
```
(function ($) {
    'use strict';
    
    var allowedMethods = ["val","text", "setVal","disable"];
    
    var ZTreeSelect = function(el, options){
    	this.$element = $(el);
    	this.options = options;
    	this.init();
    };
    
    ZTreeSelect.defaults = {
    		placeholder: "请选择",
			multiple: false,
			type: "post",
			url: null,
			data: {},
			valueKey: "value",
			textKey: "text",
			childKey: "childList",
			disabled: false,
			selectAllBtn: true,
			clearBtn: true,
			chkboxType: {"Y": "", "N": ""},
    };
    
    ZTreeSelect.prototype.init = function () {
    	var $this = this;
    	
    	this.$element.css("display", "none");
    	if(!(typeof(this.$element.attr("multiple"))=="undefined")){
    		this.options.multiple = true;
    	}
    	if(!(typeof(this.$element.attr("disabled"))=="undefined")){
    		this.options.disabled = true;
    	}
    	
    	this.$result = $('<input type="text" readonly class="ztree-select-result">');
    	this.$result.attr("placeholder", this.options.placeholder);
    	this.$element.after(this.$result);
    	if(!this.options.disabled){
    		this.$result.click(function(e){
    			$this.$container.show();
    			e.stopPropagation();
    		});
    	}else{
    		this.$result.addClass("disabled");
    	}
    	
    	this.$container = $('<div class="ztree-select-dropdown"></div>');
    	this.$container.css('display','none');
    	this.$result.after(this.$container);
    	
    	var $search = $('<span class="ztree-select-search"></span>');
    	this.$input = $('<input type="text" class="ztree-select-field">');
    	this.$input.on("input",function(){
    		var liArr = $this.$tree.find("li");
    		liArr.removeClass("hidden");
    		if(this.value == null || this.value == ""){
    			return;
    		}
    		var nodes = $this.$tree.find("a");
    		for(var i = 0; i < nodes.length; i++){
    			var nodeName = $(nodes[i]).find(".node_name").text();
    			if(nodeName != null && nodeName.indexOf(this.value) != -1){
					$(nodes[i]).addClass("matched");
				}
    		}
    		for(var i = 0; i < liArr.length; i++){
    			var matchedEl = $(liArr[i]).find("a.matched");
    			if(matchedEl.length == 0){
    				$(liArr[i]).addClass("hidden");
    			}
    		}
    		nodes.removeClass("matched");
    	});
    	$search.append(this.$input);
    	this.$container.append($search);
    	
    	//操作按钮
    	var $opreate = $('<span class="ztree-select-operate"></span>');
    	if(this.options.selectAllBtn){
    		if(this.options.multiple){
    			var $selectAllBtn = $('<span class="btn-selectAll">全选</span>');
    			$selectAllBtn.click(function(){
    				$this.treeObj.checkAllNodes(true);
    				$this.$element.empty();
					var nodes = $this.treeObj.getCheckedNodes(true);
					var result = [];
					if(nodes != null && nodes.length > 0){
						for(var i in nodes){
							$this.$element.append(new Option(nodes[i][$this.options.textKey], nodes[i][$this.options.valueKey], false, true));
							result.push(nodes[i][$this.options.textKey]);
						}
					}
					$this.$result.val(result.toString());
					$this.$result.attr("title", result.toString());
    			});
    			$opreate.append($selectAllBtn);
    		}
    	}
    	if(this.options.clearBtn){
    		var $clearBtn = $('<span class="btn-clear">清空</span>');
    		$clearBtn.click(function(){
    			$this.$element.empty();
    			$this.$result.val("");
    			$this.$result.attr("title", "");
    			if($this.options.multiple){
    				$this.treeObj.checkAllNodes(false);
    			}else{
    				var nodes = $this.treeObj.getSelectedNodes();
					if(nodes.length > 0){
						$this.treeObj.cancelSelectedNode(nodes[0]);
					}
    			}
    		});
    		$opreate.append($clearBtn);
    	}
    	if(this.options.selectAllBtn || this.options.clearBtn){
    		this.$container.append($opreate);
    	}
    	
    	//选项
    	var $options = $('<span class="ztree-select-options"></span>');
    	this.$tree = $('<ul class="ztree ztree-select"></ul>');
    	this.$tree.attr("id", this.$element.attr("id") + "_zTreeSelect");
    	$options.append(this.$tree);
    	this.$container.append($options);
    	
    	$(document).click(function(){ 
    		$this.$container.hide();
		});
    	this.$container.click(function(e){
    	    e.stopPropagation();
    	});
    	this.initzTreeSelect();
    };
    
    ZTreeSelect.prototype.initzTreeSelect = function () {
    	var $this = this;
		//初始化资源树
		var setting = {
			treeId : "",
			treeObj : null,
			async : {
				autoParam : [],
				contentType : "application/json",
				dataFilter : null,
				dataType : "json",
				enable : true,
				otherParam : this.options.data,
				type : this.options.type,
				url : this.options.url,
			},
			callback: {
				onAsyncSuccess: function(event, treeId, treeNode, msg){
					$this.treeObj.expandAll(true);
					$this.setVal($this.options.selectedValue);
				},
				beforeClick: function(treeId, treeNode){
					if($this.options.multiple){
						return false;
					}
					var nodes = $this.treeObj.getSelectedNodes();
					if(nodes.length > 0 && nodes[0][$this.options.valueKey] == treeNode[$this.options.valueKey]){
						$this.treeObj.cancelSelectedNode(nodes[0]);
						$this.$element.empty();
						$this.$result.val("");
						$this.$result.attr("title", "");
						return false;
					}
				},
				onClick: function (event, treeId, treeNode) {
					if($this.options.multiple){
						return;
					}
					$this.$element.empty();
					var result = [];
					var nodes = $this.treeObj.getSelectedNodes();
					if(nodes != null && nodes.length > 0){
						for(var i in nodes){
							$this.$element.append(new Option(nodes[i][$this.options.textKey], nodes[i][$this.options.valueKey], false, true));
							result.push(nodes[i][$this.options.textKey]);
						}
					}
					$this.$result.val(result.toString());
					$this.$result.attr("title", result.toString());
				},
				onCheck: function(){
					$this.$element.empty();
					var nodes = $this.treeObj.getCheckedNodes(true);
					var result = [];
					if(nodes != null && nodes.length > 0){
						for(var i in nodes){
							$this.$element.append(new Option(nodes[i][$this.options.textKey], nodes[i][$this.options.valueKey], false, true));
							result.push(nodes[i][$this.options.textKey]);
						}
					}
					$this.$result.val(result.toString());
					$this.$result.attr("title", result.toString());
				},
				onDblClick: function (event, treeId, treeNode, msg){
				},
			},
			check : {
				autoCheckTrigger : false,
				chkboxType : this.options.chkboxType,
				chkStyle : "checkbox",
				enable : this.options.multiple,
				nocheckInherit : false,
				chkDisabledInherit : false,
				radioType : "level"
			},
			data : {
				keep : {
					leaf : false,
					parent : false
				},
				key : {
					checked : "checked",
					children : $this.options.childKey,
					name : $this.options.textKey,
					title : "",
					url : ""
				},
				simpleData : {
					enable : false,
					idKey : "id",
					pIdKey : "parentId",
					rootPId : 0
				}
			},
			view: {
	            addHoverDom: function (treeId, node){
	            },
	            removeHoverDom: null,
	            dblClickExpand: false,
	            expandSpeed: "fast",
	            showIcon : true,
	            showLine: true,
	            showTitle : true,
	            selectedMulti: false,
	        }
		};
		this.treeObj = $.fn.zTree.init(this.$tree, setting, null);
    };
    
    ZTreeSelect.prototype.val = function(){
    	var result = [];
    	if(!this.options.multiple){
    		var nodes = this.treeObj.getSelectedNodes();
			if(nodes != null && nodes.length > 0){
				for(var i in nodes){
					result.push(nodes[i][this.options.valueKey]);
				}
			}
    	}else{
			var nodes = this.treeObj.getCheckedNodes(true);
			if(nodes != null && nodes.length > 0){
				for(var i in nodes){
					result.push(nodes[i][this.options.valueKey]);
				}
			}
    	}
    	return result.toString();
    };
    
    ZTreeSelect.prototype.text = function(){
    	var result = [];
    	if(!this.options.multiple){
    		var nodes = this.treeObj.getSelectedNodes();
			if(nodes != null && nodes.length > 0){
				for(var node in nodes){
					result.push(node[this.options.textKey]);
				}
			}
    	}else{
			var nodes = this.treeObj.getCheckedNodes(true);
			if(nodes != null && nodes.length > 0){
				for(var node in nodes){
					result.push(node[this.options.textKey]);
				}
			}
    	}
    	return result.toString();
    };

    ZTreeSelect.prototype.setVal = function(vals){
    	this.$result.empty();
    	if(vals == null || vals == ""){
    		return;
    	}
    	var nodes = this.treeObj.transformToArray(this.treeObj.getNodes());
    	if(!this.options.multiple){
    		for(var i in nodes){
    			if(vals == nodes[i][this.options.valueKey]){
    				this.$element.append(new Option(nodes[i][this.options.textKey], nodes[i][this.options.valueKey], false, true));
					this.treeObj.selectNode(nodes[i]);
					this.$result.val(nodes[i][this.options.textKey]);
					this.$result.attr("title", nodes[i][this.options.textKey]);
    			}
    		}
    	}else{
    		var valArr = vals.split(",");
    		for(var i in valArr){
    			for(var j in nodes){
    				if(valArr[i] == nodes[j][this.options.valueKey]){
						this.treeObj.checkNode(nodes[j], true, false);
    				}
    			}
    		}
    		var nodes = this.treeObj.getCheckedNodes(true);
			if(nodes != null && nodes.length > 0){
				var result = [];
				for(var i in nodes){
					this.$element.append(new Option(nodes[i][this.options.textKey], nodes[i][this.options.valueKey], false, true));
					result.push(nodes[i][this.options.textKey]);
				}
				this.$result.val(result.toString());
				this.$result.attr("title", result.toString());
			}
    	}
    };
    
    $.fn.zTreeSelect = function (option) {
    	var value,
        	args = Array.prototype.slice.call(arguments, 1);

	    this.each(function () {
	        var $this = $(this),
	            data = $this.data('zTree.select'),
	            options = $.extend({}, ZTreeSelect.defaults, $this.data(),
	                typeof option === 'object' && option);
	
	        if (typeof option === 'string') {
	            if ($.inArray(option, allowedMethods) < 0) {
	                throw new Error("Unknown method: " + option);
	            }
	
	            if (!data) {
	                return;
	            }
	
	            value = data[option].apply(data, args);
	
	            if (option === 'destroy') {
	                $this.removeData('zTree.select');
	            }
	        }
	
	        if (!data) {
	            $this.data('zTree.select', (data = new ZTreeSelect(this, options)));
	        }
	    });
	
	    return typeof value === 'undefined' ? this : value;
    }
    
})(jQuery);
```
**css**
```
.ztree-select{
    margin: 0;
    padding: 0px;
    color: #333;
}

.ztree li span.button.chk.checkbox_false_part {background-position:0 -260px}
.ztree li span.button.chk.checkbox_false_part_focus {background-position:0 -260px}

.ztree-select-result{
    border: 1px solid #E3E3E3 !important;
    height: 38px !important;
    
    background-color: #fff;
    border: 1px solid #aaa;
    border-radius: 4px;
    
    box-sizing: border-box;
    cursor: pointer;
    display: block;
    height: 28px;
    user-select: none;
    -webkit-user-select: none;
    
    line-height: 38px;
    
    box-sizing: border-box;
    list-style: none;
    margin: 0;
    padding: 2px 10px;
    width: 100%;
    
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    vertical-align: middle;
}
.ztree-select-result.disabled{
	background-color: #e9ecef;
	color:#565656;
}

.ztree-select-search{
	display: block;
    padding: 4px;
    padding-bottom:0px;
}

.ztree-select-field{
	border: 1px solid #aaa;
    padding: 4px;
    width: 100%;
    box-sizing: border-box;
    
    border: 1px solid #e3e3e3 !important;
}

.ztree-select-operate{
	padding: 4px;
	padding-bottom:0px;
	display: block;
}
.ztree-select-operate .btn-selectAll,.ztree-select-operate .btn-clear{
	text-align:center;
    border: 1px solid #e3e3e3 !important;
    padding: 3px 20px;
    margin-right: 5px;
    color: #333;
    display: inline-table;
}

.ztree-select-dropdown{
	border-top: none;
    border-top-left-radius: 0;
    border-top-right-radius: 0;
    
    border: 1px solid #e3e3e3 !important;
    box-shadow: 0 2px 2px rgba(0, 0, 0, 0.15);
    
    background-color: white;
    border: 1px solid #aaa;
    box-sizing: border-box;
    display: block;
    position: absolute;
    z-index: 1051;
    width: 100%;
}

.ztree-select-options{
	margin-top: 4px;
	display: block;
	overflow: auto;
	max-height: 350px;
}
```
