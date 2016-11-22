/**
 * Created by Administrator on 2016/10/30 0030.
 */
//当为两个参数的时候,我们实现2d变换函数属性值的读取
//当为三个参数的时候,我们实现2d变换函数属性的设置
(function (window) {
    /*transition*/
    window.cssTransform = function (obj,attr,value) {
        if(!obj.transform){
            obj.transform={};
        }
        if(arguments.length>2){//设置
            obj.transform[attr] = value;
            var text = '';
            for (var item in obj.transform) {
                switch (item){
                    case 'rotate':
                    case 'skewX':
                    case "skewY":
                    case "skew":
                        text += item+'('+obj.transform[item]+'deg) ';
                        break;
                    case 'translateX':
                    case 'translateY':
                    case 'translateZ':
                    case "translate":
                        text+=item+'('+obj.transform[item]+'px) ';
                        break;
                    case 'scale':
                    case 'scaleX':
                    case "scaleY":
                        text+=item+'('+obj.transform[item]+') ';
                        break;
                }
            }
            obj.style.transform = text;
        }else {//读取
            value = obj.transform[attr];
            if(typeof value == 'undefined'){
                //返回默认值
                if(attr == 'scale' || attr == 'scaleX' || attr == 'scaleY'){
                    return 1;
                }else {
                    return 0;
                }
            }
            return value;
        }
    };

    /*纵向滑屏*/
    window.drag = function(wrapper,index,callBack) {
        var child = wrapper.children[index];
        cssTransform(child,'translateZ',0.001);
        /*拖拽*/
        //鼠标的默认位置
        var startPoint = {X:0,Y:0};
        //元素的默认位置
        var elementPoint = {X:0,Y:0};
        //自定义一个拉伸的系数
        var step = 0;
        /*橡皮筋效果*/
        //默认开始的时间、距离
        var startTime = 0;
        var startDis = 0;
        //默认的时间、距离的差值
        var timeValue = 1;
        var disValue = 0;
        //速度
        var speed = 0;
        var isY = true;
        var isFirst = true;
        var mintop = wrapper.clientHeight - child.offsetHeight;
        var Tween ={
            SineOut: function(t,b,c,d){
                return c * Math.sin(t/d * (Math.PI/2)) + b;
            },
            BackOut: function(t,b,c,d,s){
                if (s == undefined) s = 1.70158;
                return c*((t=t/d-1)*t*((s+1)*t + s) + 1) + b;
            }
        };
        wrapper.addEventListener('touchstart',function (event) {
            mintop = wrapper.clientHeight - child.offsetHeight;
            clearInterval(child.clear);

            //鼠标的位置
            startPoint = {X:event.changedTouches[0].clientX,Y:event.changedTouches[0].clientY};

            elementPoint = {X:cssTransform(child,'translateX'),Y:cssTransform(child,'translateY')};
            /*橡皮筋效果*/
            //开始的时间、距离
            startTime = new Date().getTime();
            startDis = startPoint.Y;
            //元素的位置

            if(callBack&&callBack['start']){
                callBack['start']();
            }
            isY = true;
            isFirst = true;
        });
        wrapper.addEventListener('touchmove',function (event) {
            if(!isY){
                return;
            }

            //鼠标现在的位置
            var nowPoint = {X:0,Y:0};
            nowPoint = {X:event.changedTouches[0].clientX,Y:event.changedTouches[0].clientY};
            var disX = nowPoint.X - startPoint.X;
            var disY = nowPoint.Y - startPoint.Y;
            if(isFirst){
                isFirst = false;
                if(Math.abs(disX)>Math.abs(disY)){
                    isY = false;
                    return;
                }
            }
            var top = elementPoint.Y + (nowPoint.Y - startPoint.Y);
            //限制navList的移动范围
            if(top>0){
                step = 0.6 - top/(document.documentElement.clientHeight*3);
                var dis = step*top;
                top = dis;
            }else if(top<mintop){
                var minDis = mintop - top;
                step = 0.6 - minDis/(document.documentElement.clientHeight*3);
                top = mintop - minDis*step;
            }
            /*橡皮筋效果*/
            //结束的时间、距离
            var nowTime = new Date().getTime();
            var nowDis = nowPoint.Y;

            timeValue = nowTime - startTime;
            disValue = nowDis - startDis;
            //重新更新开始的时间、距离
            startTime = nowTime;
            startDis = nowDis;

            cssTransform(child,'translateY',top);

            if(callBack&&callBack['moving']){
                callBack['moving']();
            }
        });
        wrapper.addEventListener('touchend',function () {
            /*橡皮筋效果*/
            //速度
            speed = disValue/timeValue;

            var top = speed*500;
            //end之后移动的距离
            var translateY = cssTransform(child,'translateY');
            var target = translateY + top;
            var type="SineOut";
            //
            var time = 0;
            time = Math.abs(speed)*0.5;
            time = time<0.3?0.3:time;

            if(target>0){
                target=0;
                type = 'BackOut';
            }else if(target<mintop){
                target = mintop;
                type = 'BackOut';
            }
            move(target,time,type);
            if(callBack&&callBack['end']){
                callBack['end']();
            }

        });
        function move(target,time,type) {
            var t = 0;
            var b = cssTransform(child,'translateY');
            var c = target - b;
            var d = time/0.01;

            clearInterval(child.clear);
            child.clear = setInterval(function () {
                t++;
                if(t>d){
                    clearInterval(child.clear);
                    if(callBack&&callBack['over']){
                        callBack['over']();
                    }
                }else {
                    var T = Tween[type](t,b,c,d);
                    cssTransform(child,'translateY',T);
                    if(callBack&&callBack['moving']){
                        callBack['moving']();
                    }
                }
            },10)
        }
    };
})(window);
