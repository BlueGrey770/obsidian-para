---
type: memo
date: 2026-11-06
aliases:
  - "gou-jian-zhe-she-ji-mo-shi"
  - "构建者设计模式"
---

# 构建者设计模式

### 什么是构建者设计模式

对象如何创建？方式有很多，比如以下方式

```Java
// 无参构造，属性赋值
Member member = new Member();
member.setPhone("13312341234");
member.setName("花开小世界");
member.setGender(1);

// 有参构造
Member member1 = new Member("13312341234", "花开小世界", 1);
```

对比以上方式，也可以选择更加优雅的方式书写，比如以下代码，也是创建对象，并且赋值

```Java
Member member = Member.builder()
        .phone("13312341234")
        .name("花开小世界")
        .gender(1)
        .build();
```

在很多框架中也能看到这样写的身影，这种编写代码的方式就是构建者设计模式，好处是易于扩展，方便类的使用



### 详细的代码构成

当我们在一个实体类中添加了`@Builder`注解后，这个类就支持了构建者模式来使用这个类，我们可以查看生成后的class文件，以下代码是生成后的核心代码

```Java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.zzyl.entity;

import com.zzyl.base.BaseEntity;

public class Member extends BaseEntity {
    private String phone;
    private String name;
    private String avatar;
    private String openId;
    private int gender;
    
    // 省略了get 和  set方法

    public static MemberBuilder builder() {
        return new MemberBuilder();
    }

    public Member(final String phone, final String name, final String avatar, final String openId, final int gender) {
        this.phone = phone;
        this.name = name;
        this.avatar = avatar;
        this.openId = openId;
        this.gender = gender;
    }

    public Member() {
    }

    public static class MemberBuilder {
        private String phone;
        private String name;
        private String avatar;
        private String openId;
        private int gender;

        MemberBuilder() {
        }

        public MemberBuilder phone(final String phone) {
            this.phone = phone;
            return this;
        }

        public MemberBuilder name(final String name) {
            this.name = name;
            return this;
        }

        public MemberBuilder avatar(final String avatar) {
            this.avatar = avatar;
            return this;
        }

        public MemberBuilder openId(final String openId) {
            this.openId = openId;
            return this;
        }

        public MemberBuilder gender(final int gender) {
            this.gender = gender;
            return this;
        }

        public Member build() {
            return new Member(this.phone, this.name, this.avatar, this.openId, this.gender);
        }

        public String toString() {
            return "Member.MemberBuilder(phone=" + this.phone + ", name=" + this.name + ", avatar=" + this.avatar + ", openId=" + this.openId + ", gender=" + this.gender + ")";
        }
    }
}
```

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第三章-AI辅助快速完成登录认证 / 构建者设计模式 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/OQ2ewDabvihhjekcqohccjuxngt)
