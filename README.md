# 进销存管理系统

#### 介绍

进销存系统是为了对企业生产经营中进货、出货、批发销售、付款等全程进行（从接获订单合同开
始，进入物料采购、入库、领用到产品完工入库、交货、回收货款、支付原材料款等）跟踪（每一步都
提供详尽准确的数据）、管理（有效辅助企业解决业务管理、分销管理、存货管理、营销计划的执行和
监控、统计信息的收集等方面的业务问题）而设计的整套方案。

#### 软件环境

Springboot+MybatisPlus+SpringMvc+Freemarker+Maven

#### 软件架构

1、	基础资料 往来单位资料 货品资料 员工信息 仓库资料 计量单位 账户信息 公司信息 用户可以快速、直观地 查询所需要的数据资料。

2、	系统管理 操作员管理 系统设置 数据初始化 系统管理是整个系统的门户，在系统的安全性上起到了不可估 量的作用。各种信息要求尽量全面详细，使管理变得更轻松更有效。

3、	采购管理 新增采购订单 采购订单查询 新增采购单 采购单查询 采购退货 采购明细表 货品采购汇总表 供应商采购汇总表 采购订单完成情况 采购覆盖企业采购的各个环节。企业通过虚拟的在线货品目 录，迅速而实时的访问货品信息；通过价格和品质的比较，选定产品供应商。

4、	销售管理 新增销售订单 销售订单查询 新增销售单 销售单查询 销售退货 销售明细表 货品销售汇总表 客户销售汇总表 销售订单完成情况 销售覆盖企业销售的各个环节。通过销售订单录入与变更，跟 踪管理商品销售情况；根据货品报价和销售数量自动开出销售发票，根据发货单产生结算凭证和 收货单。

5、	库存管理 新增入库单 新增出库单 仓库调拨 库存盘点 期末提供了货品盘点、货品调价以及业务审核等 期末业务处理功能，业务期末结算为财务期末结算做了必要的铺垫作用。

6、	财务管理 付款单 收款单 其他收入 其他支出 账户查询 应付账款表-单据 应付账款表-往来单位 应收账款 表-单据 应收账款表-往来单位

# 运行指导

idea导入源码空间站顶目教程说明(Vindows版)-ssm篇：

http://mtw.so/5MHvZq 

源码地址：[http://codegym.top](http://codegym.top/)。 



#### 安装教程

环境是JDK8 

#### 使用说明

 密码都是密文加密 ，初始密码都为123456。


## 代码
CaptchaCodeFilter

```java
package com.lzj.admin.filter;


import com.fasterxml.jackson.databind.ObjectMapper;
import com.lzj.admin.model.CaptchaImageModel;
import com.lzj.admin.model.RespBean;
import com.lzj.admin.utils.StringUtil;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.session.SessionAuthenticationException;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.ServletRequestBindingException;
import org.springframework.web.bind.ServletRequestUtils;
import org.springframework.web.context.request.ServletWebRequest;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.Objects;

/**
 * 验证码过滤器 用于验证验证码是否有误
 * Security的过滤器 基于AOP的原理 在登录验证前切入验证码验证
 * @author TianTian
 * @date 2022/1/8 11:06
 */
@Component
public class CaptchaCodeFilter extends OncePerRequestFilter {
    //利用这个对象向前端传输对象 新技术 不了解
    private static ObjectMapper objectMapper = new ObjectMapper();

    /**
     * 过器实现
     * @param request
     * @param response
     * @param filterChain
     * @throws ServletException
     * @throws IOException
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
            //判断请求地址是否为login
        if (StringUtil.equals(request.getRequestURI(),"/login")){
            try {
                this.validata(new ServletWebRequest(request));
            } catch (AuthenticationException e) {
                response.setContentType("application/json;charset=UTF-8");
               response.getWriter().write((objectMapper.writeValueAsString(RespBean.error("验证码错误"))));
               return;//这里非常关键 如果不返回的话会继续执行登录验证 导致前端会收到两个Json对象
            }
        }

        filterChain.doFilter(request,response);
    }

    /**
     * 验证方法实现
     * @param request
     * @throws ServletRequestBindingException
     */
    private void validata(ServletWebRequest request) throws ServletRequestBindingException {
        HttpSession session = request.getRequest().getSession();
        //获取请求中参数值
        String captchaCode = ServletRequestUtils.getStringParameter(request.getRequest(), "captchaCode");
        //验证前端字符串是否为空
        if(StringUtil.isEmpty(captchaCode)){
            throw new SessionAuthenticationException("验证码为空");
        }
        //获取Session中存放的验证码包装类
        CaptchaImageModel captcha_key = (CaptchaImageModel) session.getAttribute("captcha_key");
        //获取验证码图片中字符串
        String code = captcha_key.getCode();
        if(Objects.isNull(captcha_key)){
            throw new SessionAuthenticationException("验证码不存在");
        }
        if (captcha_key.isExpired())
        {
            throw new SessionAuthenticationException("验证码超时");
        }

        if (!StringUtil.equals(captchaCode,code)){
            throw new SessionAuthenticationException("验证码不正确");
        }
    }
}

```


CustomerReturnListController 

```java
package com.lzj.admin.controller;


import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import com.lzj.admin.model.RespBean;
import com.lzj.admin.pojo.CustomerReturnList;
import com.lzj.admin.pojo.CustomerReturnListGoods;
import com.lzj.admin.pojo.SaleList;
import com.lzj.admin.pojo.SaleListGoods;
import com.lzj.admin.query.CustomerReturnListQuery;
import com.lzj.admin.service.CustomerReturnListService;

import com.lzj.admin.service.UserService;
import com.lzj.admin.utils.AssertUtil;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.annotation.Resource;
import java.security.Principal;
import java.util.List;
import java.util.Map;

/**
 * 客户退货单控制器
 * @author TianTian
 * @date 2022/1/19 22:58
 */
@Controller
@RequestMapping("/customerReturn")
public class CustomerReturnListController {


    @Resource
    private CustomerReturnListService customerReturnListService;


    @Resource
    private UserService userService;

    /**
     * 客户退货主页
     * @return
     */
    @RequestMapping("index")
    public String index(Model model){
        model.addAttribute("customerReturnNumber",customerReturnListService.getNextCustomerReturnNumber());
        return "customerReturn/customer_return";
    }

    @RequestMapping("save")
    @ResponseBody
    public RespBean save(CustomerReturnList customerReturnList, String goodsJson, Principal principal){
        String userName = principal.getName();
        customerReturnList.setUserId(userService.findForName(userName).getId());
        Gson gson = new Gson();
        List<CustomerReturnListGoods> slgList = gson.fromJson(goodsJson,new TypeToken<List<CustomerReturnListGoods>>(){}.getType());
        customerReturnListService.saveCustomerReturnList(customerReturnList,slgList);
        return RespBean.success("商品退货入库成功!");
    }


    /**
     * 退货单查询页
     * @return
     */
    @RequestMapping("searchPage")
    public String searchPage(){
        return "customerReturn/customer_return_search";
    }


    /**
     * 退货单列表
     * @param customerReturnListQuery
     * @return
     */
    @RequestMapping("list")
    @ResponseBody
    public Map<String,Object> customerReturnList(CustomerReturnListQuery customerReturnListQuery){
        return customerReturnListService.customerReturnList(customerReturnListQuery);
    }
    /**
     * 删除进货单记录
     * @param id
     * @return
     */
    @RequestMapping("delete")
    @ResponseBody
    public RespBean delete(Integer id){
        customerReturnListService.deleteCustomerReturn(id);
        return RespBean.success("删除成功");
    }




}

```
