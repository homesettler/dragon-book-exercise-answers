# 6.7 回填
## 6.7.1 使用回填技术的一趟式目标代码生成
非终结符号B的综合属性truelist和falselist将用来管理布尔表达式的跳转代码中的标号。
辅助函数：
	
		makelist(i) 创建一个只包含i的列表。这里i是指令数组的下标。函数makelist返回一个指向新创建的列表的指针
		merge(p1, p2)将p1和p2指向的列表进行合并，它返回的指针指向合并后的列表。
		backpatch(p, i)将i作为目标标号插入到p所指列表中的各指令中。
		
## 6.7.2 布尔表达式的回填
	1)  B->B1 || M B2   { backpatch(B1.falselist, M.instr);
						  B.truelist = merge(B1.truelist, B2.truelist);
						  B.falselist = B2.falselist;}
						  
	2) B->B1 && M B2    { backpatch(B1.truelist, M.instr);
				          B.truelist = B2.truelist;
						  B.falselist =merge(B1.falselist, B2.falselist);}
						  
	3) B->!B1           { B.truelist = B1.falselist;
						  B.falselist = B1.truelist;}
						  
	4) B->(B1)          { B.truelist = B1.truelist;
						  B.falselist = B1.falselist;}
						  
	5) B->E1 rel E2     { B.truelist = makelist(nextinstr);
						  B.falselist = makelist(nextinstr+1);
						  gen('if' E1.addr rel.op E2.addr 'goto _');
						  gen('goto _');}
						  
	6) B->true          { B.truelist = makelist(nextinstr);
						  gen('goto _');}
						  
	7) B->false         { B.falselist = makelist(nextinstr);
						  gen('goto _');}
						  
	8) M->空            { M.instr = nextinstr;}

### 注释语法分析树见课本P265
## 6.7.3 转移控制语句
