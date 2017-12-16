---
layout: post
title: Server 使用Spring来构建服务
date: 2017.11.13 20:32
tag: 后端开发
---

> 刚才一篇我们提到了如何入门后端, 我也知道一篇文章是不可能完成后端的方方面面的, 期望达到的是知道如何进行学习, 有一条清晰的学习路径, 少绕弯路. 这篇我们就来进行简历的接口开发了.

![](http://upload-images.jianshu.io/upload_images/1229762-b906bd1959a9f779.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 参考链接:

- [Server 入门后端你要学什么](http://www.jianshu.com/p/ec78451ed312)

以下内容在上述文章基础上进行, 请事先查阅.

#### jdbc 代码生成

首先我们要做的就是使用mybatis插件进行读取数据库数据并生成jdbc代码.

##### generatorConfig.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--导入属性配置-->
    <properties resource="datasource.properties"></properties>

    <!--指定特定数据库的jdbc驱动jar包的位置-->
    <classPathEntry location="${db.driverLocation}"/>

    <context id="default" targetRuntime="MyBatis3">

        <!-- optional，旨在创建class时，对注释进行控制 -->
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--jdbc的数据库连接 -->
        <jdbcConnection
                driverClass="${db.driverClassName}"
                connectionURL="${db.url}"
                userId="${db.username}"
                password="${db.password}">
        </jdbcConnection>


        <!-- 非必需，类型处理器，在数据库类型和java类型之间的转换控制-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>


        <!-- Model模型生成器,用来生成含有主键key的类，记录类 以及查询Example类
            targetPackage     指定生成的model生成所在的包名
            targetProject     指定在该项目下所在的路径
        -->
        <!--<javaModelGenerator targetPackage="com.mmall.pojo" targetProject=".\src\main\java">-->
        <javaModelGenerator targetPackage="com.resume.pojo" targetProject="./src/main/java">
            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="false"/>
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
            <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>

        <!--mapper映射文件生成所在的目录 为每一个数据库的表生成对应的SqlMap文件 -->
        <!--<sqlMapGenerator targetPackage="mappers" targetProject=".\src\main\resources">-->
        <sqlMapGenerator targetPackage="mappers" targetProject="./src/main/resources">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码
                type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
                type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
                type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
        -->

        <!-- targetPackage：mapper接口dao生成的位置 -->
        <!--<javaClientGenerator type="XMLMAPPER" targetPackage="com.resume.dao" targetProject=".\src\main\java">-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.resume.dao" targetProject="./src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>

        <!--<table tableName="resume_profile" domainObjectName="Profile" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>-->
        <!--<table tableName="resume_profile_interest" domainObjectName="ProfileInterest" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>-->
        <!--<table tableName="resume_profile_education" domainObjectName="ProfileEducation" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>-->
        <table tableName="resume_profile_social" domainObjectName="ProfileSocial" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>

    </context>
</generatorConfiguration>
```

配置完成后执行插件进行生成:

![](http://upload-images.jianshu.io/upload_images/1229762-385a6be2262902f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


mybatis 就是类似封装了fmdb, 直接和数据库打交道. 通过插件我们生成了dao / pojo / mapper的文件.

##### dao

```
package com.resume.dao;

import com.resume.pojo.ProfileSocial;

import java.util.List;

public interface ProfileSocialMapper {
    int deleteByPrimaryKey(Integer id);

    int insert(ProfileSocial record);

    int insertSelective(ProfileSocial record);

    ProfileSocial selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(ProfileSocial record);

    int updateByPrimaryKey(ProfileSocial record);

    List<ProfileSocial> selectAllSocial();

}
```

接口文件, 用于调用mapper的jdbc.

##### mapper

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.resume.dao.ProfileSocialMapper" >
  <resultMap id="BaseResultMap" type="com.resume.pojo.ProfileSocial" >
    <constructor >
      <idArg column="id" jdbcType="INTEGER" javaType="java.lang.Integer" />
      <arg column="src" jdbcType="VARCHAR" javaType="java.lang.String" />
      <arg column="href" jdbcType="VARCHAR" javaType="java.lang.String" />
    </constructor>
  </resultMap>
  <sql id="Base_Column_List" >
    id, src, href
  </sql>
  <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Integer" >
    select 
    <include refid="Base_Column_List" />
    from resume_profile_social
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer" >
    delete from resume_profile_social
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="com.resume.pojo.ProfileSocial" >
    insert into resume_profile_social (id, src, href
      )
    values (#{id,jdbcType=INTEGER}, #{src,jdbcType=VARCHAR}, #{href,jdbcType=VARCHAR}
      )
  </insert>
  <insert id="insertSelective" parameterType="com.resume.pojo.ProfileSocial" >
    insert into resume_profile_social
    <trim prefix="(" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        id,
      </if>
      <if test="src != null" >
        src,
      </if>
      <if test="href != null" >
        href,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        #{id,jdbcType=INTEGER},
      </if>
      <if test="src != null" >
        #{src,jdbcType=VARCHAR},
      </if>
      <if test="href != null" >
        #{href,jdbcType=VARCHAR},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="com.resume.pojo.ProfileSocial" >
    update resume_profile_social
    <set >
      <if test="src != null" >
        src = #{src,jdbcType=VARCHAR},
      </if>
      <if test="href != null" >
        href = #{href,jdbcType=VARCHAR},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="com.resume.pojo.ProfileSocial" >
    update resume_profile_social
    set src = #{src,jdbcType=VARCHAR},
      href = #{href,jdbcType=VARCHAR}
    where id = #{id,jdbcType=INTEGER}
  </update>
  <select id="selectAllSocial" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from resume_profile_social
  </select>
</mapper>
```
封装了一些数据库的逻辑的操作.


pojo 就是javabean, 我们来看下一个简单的javabean是个什么东西吧.

```

package com.resume.pojo;

public class ProfileInterest {
    private Integer id;

    private String interest;

    public ProfileInterest(Integer id, String interest) {
        this.id = id;
        this.interest = interest;
    }

    public ProfileInterest() {
        super();
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getInterest() {
        return interest;
    }

    public void setInterest(String interest) {
        this.interest = interest == null ? null : interest.trim();
    }
}
```

我们可以看到和我们model其实是差不多的, 只不过oc的代码看起来比较简洁罢了.

可以得知我们通过dao操控mapper并将数据存储至pojo返回前端.

#### 高复用响应

##### ResponseCode

```
package com.resume.common;

/**
 * Created by zhushuangquan on 08/11/2017.
 */
public enum  ResponseCode {

    SUCCESS(0, "SUCCESS"),
    ERROR(1, "ERROR"),
    NEED_LOGIN(10, "NEED_LOGIN"),
    ILLEGAL_ARGUMENT(2, "ILLEGAL_ARGUMENT");

    private final int code;
    private final String desc;

    ResponseCode(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    public  int getCode() {
        return code;
    }
    public  String getDesc() {
        return desc;
    }
}

```

##### ServerResponse

```
package com.resume.common;

import org.codehaus.jackson.annotate.JsonIgnore;
import org.codehaus.jackson.map.annotate.JsonSerialize;

import java.io.Serializable;

/**
 * Created by zhushuangquan on 08/11/2017.
 */
@JsonSerialize(include = JsonSerialize.Inclusion.NON_NULL)
public class ServerResponse<T> implements Serializable {

    private int status;
    private String msg;
    private T data;

    private ServerResponse(int status) {
        this.status = status;
    }
    private ServerResponse(int status,  T data) {
        this.status = status;
        this.data = data;
    }
    private ServerResponse(int status, String msg, T data) {
        this.status = status;
        this.msg = msg;
        this.data = data;
    }
    private ServerResponse(int status, String msg) {
        this.status = status;
        this.msg = msg;
    }

    @JsonIgnore
    public boolean isSuccess() {
        return this.status == ResponseCode.SUCCESS.getCode();
    }

    public int getStatus() {
        return status;
    }

    public T getData() {
        return data;
    }

    public String getMsg() {
        return msg;
    }

    public static <T> ServerResponse<T> createBySuccess() {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode());
    }

    public static <T> ServerResponse<T> createBySuccessMessage(String msg) {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode(), msg);
    }

    public static <T> ServerResponse<T> createBySuccess(T data) {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode(), data);
    }

    public static <T> ServerResponse<T> createBySuccess(String msg, T data) {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode(), msg, data);
    }

    public static <T> ServerResponse<T> createByError() {
        return new ServerResponse<T>(ResponseCode.ERROR.getCode(), ResponseCode.ERROR.getDesc());
    }

    public static <T> ServerResponse<T> createByErrorMessage(String errorMessage) {
        return new ServerResponse<T>(ResponseCode.ERROR.getCode(), errorMessage);
    }

    public static <T> ServerResponse<T> createByErrorCodeMessage(int errorCode, String errorMessage) {
        return new ServerResponse<T>(errorCode, errorMessage);
    }
}


```

这里有两个注解要说下`@JsonSerialize(include = JsonSerialize.Inclusion.NON_NULL)` 将对象转换成json序列化的注解, 以及`@JsonIgnore`忽略转化的注解.

#### 接口开发

终于到接口开发这个激动人心的时刻了, 我们还是来回顾一下我们需要制作接口的图示:

![](http://upload-images.jianshu.io/upload_images/1229762-a3d8a0c34c76e69e.png?imageMogr2/auto-orient/strip)

##### controller

首先我们需要新建一个控制器, 取名为 PortalController, 针对前台的控制器.

```
package com.resume.controller;

import com.resume.common.ServerResponse;
import com.resume.service.IPortalService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletResponse;

/**
 * Created by zhushuangquan on 08/11/2017.
 */

@Controller
@RequestMapping("/portal/")
public class PortalController {

    @Autowired
    private IPortalService iPortalService;

    @RequestMapping("fetch_profile.do")
    @ResponseBody
    public ServerResponse fetchProfile(HttpServletResponse response) {
        response.setHeader("Access-Control-Allow-Origin", "*");
        return iPortalService.fetchProfile();
    }
}

```

这里我们看到很多springmvc和spring的注解, `@Controller`控制器注解, `@RequestMapping`路由注解, `@Autowired`自动装配注解, `@ResponseBody` 响应体注解, 用于将返回的对象通过json传给前台解析数据. 其中service就是我们接下来要讲到的服务层, 而`        response.setHeader("Access-Control-Allow-Origin", "*");
`这句仅仅是用来跨域使用的.

##### service

```
package com.resume.service;

import com.resume.common.ServerResponse;
import com.resume.pojo.Profile;

/**
 * Created by zhushuangquan on 09/11/2017.
 */
public interface IPortalService {

    ServerResponse<Profile> fetchProfile();
}
```
```
package com.resume.service.impl;

import com.google.common.collect.Lists;
import com.resume.common.ServerResponse;
import com.resume.dao.ProfileEducationMapper;
import com.resume.dao.ProfileInterestMapper;
import com.resume.dao.ProfileMapper;
import com.resume.dao.ProfileSocialMapper;
import com.resume.pojo.Profile;
import com.resume.pojo.ProfileEducation;
import com.resume.pojo.ProfileInterest;
import com.resume.pojo.ProfileSocial;
import com.resume.service.IPortalService;
import com.resume.vo.ProfileEducationVo;
import com.resume.vo.ProfileInterestVo;
import com.resume.vo.ProfileSocialVo;
import com.resume.vo.ProfileVo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * Created by zhushuangquan on 09/11/2017.
 */

@Service("iPortalService")
public class PortalServiceImpl implements IPortalService {

    @Autowired
    private ProfileMapper profileMapper;

    @Autowired
    private ProfileSocialMapper profileSocialMapper;

    @Autowired
    private ProfileInterestMapper profileInterestMapper;

    @Autowired
    private ProfileEducationMapper profileEducationMapper;

    public ServerResponse fetchProfile() {

        ProfileVo profileVo = assembleProfileVo(profileMapper.selectAll());
        List<ProfileSocial> profileSocialList = profileSocialMapper.selectAllSocial();
        List<ProfileSocialVo> profileSocialVoList = Lists.newArrayList();
        List<ProfileInterest> profileInterestList = profileInterestMapper.selectAllInterest();
        List<ProfileInterestVo> profileInterestVoList = Lists.newArrayList();
        List<ProfileEducation> profileEducationList = profileEducationMapper.selectAllEducation();
        List<ProfileEducationVo> profileEducationVoList = Lists.newArrayList();
        for (ProfileSocial profileSocial: profileSocialList) {
            ProfileSocialVo profileSocialVo = assembleProfileSocialVo(profileSocial);
            profileSocialVoList.add(profileSocialVo);
        }
        for (ProfileInterest profileInterest: profileInterestList) {
            ProfileInterestVo profileInterestVo = assembleProfileInterestVo(profileInterest);
            profileInterestVoList.add(profileInterestVo);
        }
        for (ProfileEducation profileEducation: profileEducationList) {
            ProfileEducationVo profileEducationVo = assembleProfileEducationVo(profileEducation);
            profileEducationVoList.add(profileEducationVo);
        }
        profileVo.setProfileSocialList(profileSocialVoList);
        profileVo.setProfileInterestList(profileInterestVoList);
        profileVo.setProfileEducationList(profileEducationVoList);
        return ServerResponse.createBySuccess(profileVo);
    }

    private ProfileVo assembleProfileVo(Profile profile) {
        ProfileVo profileVo = new ProfileVo();
        profileVo.setProfileImage(profile.getProfileImage());
        profileVo.setProfileInterestTitle(profile.getProfileInterestTitle());
        profileVo.setProfileCareer(profile.getProfileCareer());
        profileVo.setProfileName(profile.getProfileName());
        profileVo.setProfileEducationTitle(profile.getProfileEducationTitle());
        profileVo.setProfileLocation(profile.getProfileLocation());
        profileVo.setProfileSummaryTitle(profile.getProfileSummaryTitle());
        profileVo.setProfileSummaryDescription(profile.getProfileSummaryDescription());
        return profileVo;
    }

    private ProfileSocialVo assembleProfileSocialVo(ProfileSocial profileSocial) {
        ProfileSocialVo profileSocialVo = new ProfileSocialVo();
        profileSocialVo.setSrc(profileSocial.getSrc());
        profileSocialVo.setHref(profileSocial.getHref());
        return profileSocialVo;
    }

    private ProfileInterestVo assembleProfileInterestVo(ProfileInterest profileInterest) {
        ProfileInterestVo profileInterestVo = new ProfileInterestVo();
        profileInterestVo.setInterest(profileInterest.getInterest());
        return profileInterestVo;
    }

    private ProfileEducationVo assembleProfileEducationVo(ProfileEducation profileEducation) {
        ProfileEducationVo profileEducationVo = new ProfileEducationVo();
        profileEducationVo.setMajor(profileEducation.getMajor());
        profileEducationVo.setSchool(profileEducation.getSchool());
        profileEducationVo.setYear(profileEducation.getYear());
        return profileEducationVo;
    }
}


```
service层是整个业务逻辑的核心, 如何使用dao进行获取数据, 如何通过装配vo封装数据, 并返回给controller. vo和pojo其实是一样的东西, 也就是javabean, 用于封装数据格式的.

通过读表及封装返回后, 我们通过访问接口: [http://localhost:8080/portal/fetch_profile.do](http://localhost:8080/portal/fetch_profile.do), 就能够得到接口返回的数据啦, 这里讲的比较简单, 知识一个select的使用, 之后我会更详细的讲述增删改查整个流程并制作简历的后台页面, 敬请期待!

好了 我们来看看返回的数据吧.

```
/* http://localhost:8080/portal/fetch_profile.do */
{
    "status": 0,
    "data": {
        "profileImage": "http://upload-images.jianshu.io/upload_images/1229762-23e162b9bd6b9c39.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240",
        "profileName": "Shuangquan Zhu",
        "profileCareer": "Designer / Developer",
        "profileLocation": "Shanghai",
        "profileSocialList": [
            {
                "src": "http://upload-images.jianshu.io/upload_images/1229762-877e3e5c2260bcf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240",
                "href": "https://github.com/coderzsq"
            },
            {
                "src": "http://upload-images.jianshu.io/upload_images/1229762-f6525252f3e8387b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240",
                "href": "http://www.jianshu.com/u/9d7fad1a4693"
            },
            {
                "src": "http://upload-images.jianshu.io/upload_images/1229762-a69614a97de93f36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240",
                "href": "http://upload-images.jianshu.io/upload_images/1229762-453920a3f4eedcd8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240"
            },
            {
                "src": "http://upload-images.jianshu.io/upload_images/1229762-91b0949aec719aab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240",
                "href": "http://coderzsq.github.io/"
            }
        ],
        "profileSummaryTitle": "Summary",
        "profileSummaryDescription": "Shuangquan Zhu is a professional developer who focuses on iOS now. He has strong knowledge of Objective-C, Swift and Javascript. With these skills, he created quite a few quickly developer tools. He also leads the J1 iOS team to promote the project process.\\nHe crazy loves playing basketball with friends in spare time, He also loves traveling, writing and listening music. He is always willing to try new things, and keeping to learn from them.",
        "profileInterestTitle": "Interest",
        "profileInterestList": [
            {
                "interest": "Learn about high tech"
            },
            {
                "interest": "Play basketball"
            },
            {
                "interest": "Listen to music"
            },
            {
                "interest": "Apple products"
            }
        ],
        "profileEducationTitle": "Education",
        "profileEducationList": [
            {
                "major": "Business Management",
                "school": "East China University of Science and Technology",
                "year": "2016"
            },
            {
                "major": "Customs and International Freight",
                "school": "Shanghai Maritime Academy",
                "year": "2013"
            }
        ]
    }
}
```
连续更新了两篇, 其实写一个后端并没有想象中那么难, 当然要学深要走的路还很长, 这不才一个月的水准已经相当不错了, 当然诸如sql注入, 横向越权, 纵向越权, 读写分离, 缓存等知识, 还需要接下来去学习.

我感觉, 学习编程, 思想最重要, 你了解一个事情背后的原理比你记得多少api不知道强多少!!!

##### About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 项目源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.maven.java)

