# ssmcategory
SSM整合练习二  分页 增删改查



顺序
<!--
 * @Author: your name
 * @Date: 2020-08-13 10:32:06
 * @LastEditTime: 2020-08-13 11:57:09
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinede:\ssm.md
-->
CRUD

#### 1.建立数据库
``` sql
CREATE TABLE `category_` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=108 DEFAULT CHARSET=utf8;
 ```
#### 2.Category.java(pojo)：getter setter toString
``` java
private int id;
private String name;
 ```
 #### 3.Page.java(util) getter setter
 ``` java
 	int start=0;	//当前页开始
	int count = 5;	//当前页总数
	int last = 0;	//最后页
	//最后页开始判断
	public void caculateLast(int total) {
	    // 假设总数是50，是能够被5整除的，那么最后一页的开始就是45
	    if (0 == total % count)
	        last = total - count;
	    // 假设总数是51，不能够被5整除的，那么最后一页的开始就是50
	    else
	        last = total - total % count;		
	}
 ```
#### 4.CategoryMapper.java(mapper):interface
``` java
    public int add(Category category);  //增
 
    public void delete(int id);         //删
 
    public Category get(int id);        //根据id查 
 
    public int update(Category category); //改
 
    public List<Category> list();           //查全部
    
    public List<Category> list(Page page);  //查本页
  
    public int total();                     //总数
 ```
 #### 5.Category.xml(mapper)
 ``` java
    <mapper namespace="com.how2java.mapper.CategoryMapper">
	    <insert id="add" parameterType="Category" >
	        insert into category_ ( name ) values (#{name})    
	    </insert>
	    
	    <delete id="delete" parameterType="Category" >
	        delete from category_ where id= #{id}   
	    </delete>
	    
	    <select id="get" parameterType="_int" resultType="Category">
	        select * from   category_  where id= #{id}    
	    </select>

	    <update id="update" parameterType="Category" >
	        update category_ set name=#{name} where id=#{id}    
	    </update>
	    <select id="list" resultType="Category">
	        select * from   category_ order by id asc     
	        <if test="start!=null and count!=null">
                    limit #{start},#{count}
            </if>
            
	    </select>
	    <select id="total" resultType="int">
	        select count(*) from   category_       
	    </select>	    	    
	</mapper>
  ```
#### 6.CategoryService.java(service):interface
``` java
        List<Category> list();          //查
	int total();                    //总数
	List<Category> list(Page page); //查本页
	void add(Category c);           //增
	void update(Category c);        //改
	void delete(Category c);        //删
	Category get(int id);           //根据id查
 ```
#### 7.web.xml(web)
``` java
    //   A/t 加 / 添加spring配置
    <param-value>classpath:applicationContext.xml</param-value>

    //      添加springmvc配置
    <param-value>classpath:springMVC.xml</param-value>
    <url-pattern>/</url-parrent>

    //过滤器
    <filter>  
        <filter-name>CharacterEncodingFilter</filter-name>  
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
        <init-param>  
            <param-name>encoding</param-name>  
            <param-value>utf-8</param-value>  
        </init-param>  
    </filter>  
    <filter-mapping>  
        <filter-name>CharacterEncodingFilter</filter-name>  
        <url-pattern>/ *</url-pattern>  
    </filter-mapping>     
 ```
 #### 8.applicationContext.xml(src):spring
``` java
    //扫描service
        <context:annotation-config />
	<context:component-scan base-package="com.how2java.service" />

    //配置数据库连接池
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
		<!-- 基本属性 url、user、password -->
        <property name="url" value="jdbc:mysql://localhost:3306/how2java?characterEncoding=UTF-8" />
        <property name="username" value="root" />
        <property name="password" value="111111" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />

        <!-- 配置初始化大小、最小、最大 -->
        <property name="initialSize" value="3" />
        <property name="minIdle" value="3" />
        <property name="maxActive" value="20" />

        <!-- 配置获取连接等待超时的时间 -->
        <property name="maxWait" value="60000" />

        <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
        <property name="timeBetweenEvictionRunsMillis" value="60000" />

        <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
        <property name="minEvictableIdleTimeMillis" value="300000" />

        <property name="validationQuery" value="SELECT 1" />
        <property name="testWhileIdle" value="true" />
        <property name="testOnBorrow" value="false" />
        <property name="testOnReturn" value="false" />

        <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
        <property name="poolPreparedStatements" value="true" />
        <property name="maxPoolPreparedStatementPerConnectionSize" value="20" />
    </bean>

    //数据库关联mapper下xml文件
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="typeAliasesPackage" value="com.how2java.pojo" />
		<property name="dataSource" ref="dataSource"/>
		<property name="mapperLocations" value="classpath:com/how2java/mapper/ *.xml"/>
    </bean>
    
    //扫描mapper
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.how2java.mapper"/>
    </bean>
 ```
 #### 9.CategoryServiceImpl.java(service.impl)
``` java
@Service     //声明Service
public class CategoryServiceImpl  implements CategoryService{
	@Autowired        //自动装配CategoryMapper.java(mapper)
	CategoryMapper categoryMapper;  //mapper
	
	
	public List<Category> list(){
		return categoryMapper.list();
	}


	@Override
	public List<Category> list(Page page) {
		// TODO Auto-generated method stub
		return categoryMapper.list(page);
	}


	@Override
	public int total() {
		return categoryMapper.total();
	}


	@Override
	public void add(Category c) {
		categoryMapper.add(c);
		
	}


	@Override
	public void update(Category c) {
		categoryMapper.update(c);
	}


	@Override
	public void delete(Category c) {
		categoryMapper.delete(c.getId());
	}


	@Override
	public Category get(int id) {
		// TODO Auto-generated method stub
		return categoryMapper.get(id);
	};

}

 ```
 #### 10.springMVC.xml(src)
 ``` java
 <context:annotation-config/>
    //扫描controller
    <context:component-scan base-package="com.how2java.controller">
          <context:include-filter type="annotation" 
          expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    //注解驱动
    <mvc:annotation-driven />
    //访问静态资源
    <mvc:default-servlet-handler />
    //视图解析器
    <bean
        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass"
            value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
  ```
#### 11.CategoryController.java(controller)
``` java
@Controller                 //声明控制器
@RequestMapping("")
public class CategoryController {
	@Autowired
	CategoryService categoryService;  //service

	@RequestMapping("listCategory")
	public ModelAndView listCategory(Page page){
	
		ModelAndView mav = new ModelAndView();
		List<Category> cs= categoryService.list(page);
		int total = categoryService.total();
		
		page.caculateLast(total);
		
		// 放入转发参数
		mav.addObject("cs", cs);
		// 放入jsp路径
		mav.setViewName("listCategory");
		return mav;
	}
	
	@RequestMapping("addCategory")
	public ModelAndView addCategory(Category category){
		categoryService.add(category);
		ModelAndView mav = new ModelAndView("redirect:/listCategory");
	    return mav;
	}	
	@RequestMapping("deleteCategory")
	public ModelAndView deleteCategory(Category category){
		categoryService.delete(category);
		ModelAndView mav = new ModelAndView("redirect:/listCategory");
		return mav;
	}	
	@RequestMapping("editCategory")
	public ModelAndView editCategory(Category category){
		Category c= categoryService.get(category.getId());
		ModelAndView mav = new ModelAndView("editCategory");
		mav.addObject("c", c);
		return mav;
	}	
	@RequestMapping("updateCategory")
	public ModelAndView updateCategory(Category category){
		categoryService.update(category);
		ModelAndView mav = new ModelAndView("redirect:/listCategory");
		return mav;
	}	

}
 ```  
 #### 12.listCategory.jsp(jsp)
 ``` java
 <%@ page  import="java.util.*"%>
 <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
 <div style="width:500px;margin:0px auto;text-align:center">
	<table align='center' border='1' cellspacing='0'>
	    <tr>
	        <td>id</td>
	        <td>name</td>
	        <td>编辑</td>
	        <td>删除</td>
	    </tr>
	    <c:forEach items="${cs}" var="c" varStatus="st">
	        <tr>
	            <td>${c.id}</td>
	            <td>${c.name}</td>
	            <td><a href="editCategory?id=${c.id}">编辑</a></td>
	            <td><a href="deleteCategory?id=${c.id}">删除</a></td>
	        </tr>
	    </c:forEach>
	</table>
	<div style="text-align:center">
		<a href="?start=0">首  页</a>
		<a href="?start=${page.start-page.count}">上一页</a>
		<a href="?start=${page.start+page.count}">下一页</a>
		<a href="?start=${page.last}">末  页</a>
	</div>
	
	
	<div style="text-align:center;margin-top:40px">
		
		<form method="post" action="addCategory">
			分类名称： <input name="name" value="" type="text"> <br><br>
			<input type="submit" value="增加分类">
		</form>

	</div>	
 </div>
  ```
#### 13.editCategory.jsp(jsp)
``` java
<%@ page import="java.util.*"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
 
 <div style="width:500px;margin:0px auto;text-align:center">
	
	
	<div style="text-align:center;margin-top:40px">
		
		<form method="post" action="updateCategory">
			分类名称： <input name="name" value="${c.name}" type="text"> <br><br>
			<input type="hidden" value="${c.id}" name="id">
			<input type="submit" value="增加分类">
		</form>

	</div>	
 </div>
 ```
