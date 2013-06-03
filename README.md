flowjs
======

一个开放的，面向业务流程的，可灵活扩展和定制的前端业务开发框架。

框架思路
-------

互联网产品的特点就是快速的迭代更新，这就对前端的响应速度提出了挑战。flowjs的思路，就是通过抽象变化的产品形态背后相对稳定的业务流程，来以不变应万变！

框架由Flow（流程）和Step（步骤）两个类组成。Flow负责定义一个业务逻辑的流程，Step负责定义流程中的一个步骤。
这与一个流程图很类似，未来我们会提供工具来实现流程图与代码的互相转换。这样就可以实现可视化的流程制定。

Step类需要明确的定义本步骤所需要的参数。

基于同一个Flow，可以通过实现不同的Step类的子类来实现扩展与定制。

开发者可以贡献抽象的Flow，也可以贡献实现好的Step，达到开放、共享的目的。

Flow定义
-------

    define(function(require, exports, module) {
        var Class = Flowjs.Class;
        var Flow = Flowjs.Flow;
        var CommonFocusFlow = Class({
            extend:Flow,
            construct:function(options){
                this.callsuper(options);
                this._addStep('step1',require('./stepdefinition/step1'));
                this._addStep('step2',require('./stepdefinition/step2'));
                this._addStep('step3',require('./stepdefinition/step3'));
                this._addStep('step4',require('./stepdefinition/step4'));
                this._addStep('step5',require('./stepdefinition/step5'));
                this._addStep('step6',require('./stepdefinition/step6'));
            },
            methods:{
                //初始化流程
                start:function(){
                    var _this = this;
                    this._go('step1');
                    this._go('step2');
                    this._go('step3');
                    this._go('step4');
                    this._go('step5',null,{
                        cases:{
                            '1':function(){
                                _this._go('step1');
                            },
                            '2':function(){
                                _this._go('step6');
                            }
                        },defaultCase:function(){
                            _this._go('step4');
                        }
                    });
                }
            }
        });

        module.exports = CommonFocusFlow;
    });

以上流程首先会顺序执行 1 -> 2 -> 3 -> 4 -> 5

step5是一个条件判断的步骤，这里会进行判断

如果结果为1，则继续 1 -> 2 -> 3 -> 4 -> 5

如果结果为2，则执行 6

其余情况，则继续 4 -> 5

Step定义
-------

基类的定义

    define(function(require,exports,module){
        var Class = Flowjs.Class;
        var Step = Flowjs.Step;
        var Next = Class({
            extend:Step,
            isAbstract:true,
            construct:function(options){
                this.callsuper(options);
            },
            methods:{
                _describeData:function(){
                    return {
                        input:{
                            frames:{
                                type:'object'
                            },
                            curr:{
                                type:'number'
                            }
                        },
                        output:{
                            curr:{
                                type:'number'
                            }
                        }
                    };
                }
            }
        });
        
        module.exports = Next;
    });



实现类的定义

    define(function(require,exports,module){
        module.exports = {
            methods:{
                _process:function(data,callback){
                    var total = data.frames.length;
                    var curr = data.curr + 1;
                    if(curr == total){
                        curr = 0;
                    }
                    callback(null,{curr:curr});
                }
            }
        };
    });


以上定义了一个步骤，要求输入的数据对象结构为：{curr:1,frames:{}}；输出的数据对象结果为{curr:2}

API
---------

Flow

    init：function(){}

        抽象方法，子类需要实现该方法来定义流程。

    implement：function(stepName,options){}

        公共方法。实现流程中的一个步骤。

        stepName

            步骤名

        options.construct

            步骤类的构造函数

        options.methods

            步骤类的方法

Step

Condition

Input

注意事项
---------

步骤与步骤之间尽可能的减少依赖

每一个步骤只能调用一次callback通知框架步骤完成

v0.1.0发布
---------

初始版本

v0.2.0发布
---------

更新内容

    流程初始化接口由 start 修改为 init 
    抽象类的定义需要显式通过abstract属性来指定
    Class方法加入对抽象方法实现的检查

升级指导

    改为调用init方法启动流程初始化
    所有抽象类都需要加入abstract:true来显式指明该类是抽象类
        extend:Step,
        construct:function(options){
            options = options || {};
            this.callsuper(options);
            this._cases = options.cases || {};
            this._default = options.defaultCase;
        },
        isAbstract:true,
        methods:{