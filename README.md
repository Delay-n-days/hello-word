# Saturday, October 10, 2020
# 添加一个子菜单

### 1.在静态数组中添加子菜单名称(把其中一个备用改为“子菜单测试1”)
```c
const char * dis_menunamec[]=
{
    "CAN仪表故障诊断",
    "..."
    "子菜单测试1"
}
```
注: `MenuNum` 变量是子菜单数量，如要添加子菜单而不是替换请修改。
### 2.在 `Dis_MenuChild` 中添加子菜单显示函数，用于显示子菜单
```c
void Dis_MenuChild(void) 
{


    switch (curmenuindex)
    {
        case 0:  /* CAN仪表 */
        case 1:  /* 前模块 */
        case 2:  /* 顶模块 */
            {
                Dis_Check();          
                break;
            }         
        case 3:
            {
                RA8875_SetFrontColor(CL_CYAN); 
                Dis_UpdateTime();
                RA8875_SetFrontColor(CL_WHITE);
                break;
            }
        ...
        case 15:
            {
                //子菜单测试1显示函数
                break;
            }     
    }
```
### 3.编写子菜单显示函数`Dis_TEST_Info`
```C
const char * TESTInfoStr[] =  //菜单项名称
{
    "test电压",
    "test电流",
    "test状态",
    "test故障代码",
};
void Dis_TEST_Info(void) 
{
    static uint16 start_index = 0;
    SubMenu_Type submenu[sizeof(DCF2InfoStr) / sizeof(char *)] = {0};
    float  ftemp = 0.0;
    int16  stemp = 0;
    uint16 utemp = 0;
    uint16 cnt = 0;
    int16  i = 0;
   // test电压
	stemp = gAppRecvCanBuf[IDX_0x185C8A7F].data[1]|gAppRecvCanBuf[IDX_0x185C8A7F].data[2];//检测报文
	stemp = stemp-3000;//偏移量
	submenu[cnt].valueType = FLOAT_TYPE;//类型
	submenu[cnt].value = (uint32_t)(stemp);//值
	submenu[cnt].unit = "V";//单位
	submenu[cnt].float_len = 1; //小数点后位数
	cnt ++;
   // test电流
	stemp = gAppRecvCanBuf[IDX_0x185C8A7F].data[3]|gAppRecvCanBuf[IDX_0x185C8A7F].data[4];
	stemp = stemp-3000;
	submenu[cnt].valueType = FLOAT_TYPE;
	submenu[cnt].value = (uint32_t)(stemp);
	submenu[cnt].unit = "A";
	submenu[cnt].float_len = 1; 
	cnt ++;
   //test状态
	utemp = ((gAppRecvCanBuf[IDX_0x185C8A7F].data[6])&0x0F); 
	submenu[cnt].valueType = STRING_TYPE;
	 if(utemp == 1)
		submenu[cnt].value = (uint32_t)("准备");
	else if(utemp == 3)
		submenu[cnt].value = (uint32_t)("工作");         
	else if(utemp == 4)
		submenu[cnt].value = (uint32_t)("超载"); 
	else if(utemp == 8)
		submenu[cnt].value = (uint32_t)("故障");     
	else
		submenu[cnt].value = (uint32_t)("  ");
	submenu[cnt].unit = "";
	submenu[cnt].float_len = 0; 
	cnt ++;
    // test故障代码
	stemp = gAppRecvCanBuf[IDX_0x185C8A7F].data[7];
	submenu[cnt].valueType = INT32_TYPE;
	submenu[cnt].value = (uint32_t)(stemp);
	submenu[cnt].unit = "";
	submenu[cnt].float_len = 0; 
	cnt ++;
    //显示函数
    Dis_MenuString_njjl(40,100,DCF2InfoStr,sizeof(DCF2InfoStr) / 4,
    &start_index,submenu);
}
```
注：IDX_0x185C8A7F为我在添加CAN报文显示报警符号片时添加的ID，添加过程详见"新建符号片报文.pdf"

最后，由于该函数与`Dis_MenuChild`在同一文件中，且在`Dis_MenuChild`上方，故此函数无需声明，
### 4.在`Init_Menu`中撤销备用，否则按下确认键会没有反应（把ChildMenu9Pro 改为 ChildMenuPro）
```C
void Init_Menu(void) 
{

    /* 初始化主菜单 */
    SysMenu[0].MenuProduce=ChildMenuPro;
    SysMenu[1].MenuProduce=ChildMenuPro;
    SysMenu[2].MenuProduce=ChildMenuPro;

    SysMenu[3].MenuProduce=ChildMenu7Pro;	/* 备用 */
    SysMenu[4].MenuProduce=ChildMenu8Pro;
    SysMenu[5].MenuProduce=ChildMenuPro;

    SysMenu[6].MenuProduce=ChildMenuPro;
    SysMenu[7].MenuProduce=ChildMenuPro;
    SysMenu[8].MenuProduce=ChildMenuPro;	


    SysMenu[9].MenuProduce =ChildMenuPro;
    SysMenu[10].MenuProduce=ChildMenuPro;
    SysMenu[11].MenuProduce=ChildMenuPro;
    SysMenu[12].MenuProduce=ChildMenuPro;
    SysMenu[13].MenuProduce=ChildMenuPro;	
    SysMenu[14].MenuProduce=ChildMenuPro;   
    SysMenu[15].MenuProduce=ChildMenuPro; /* 备用 */  //子菜单测试1是15号 	
    SysMenu[16].MenuProduce=ChildMenu9Pro;   
    SysMenu[17].MenuProduce=ChildMenu9Pro; 
}
```
