#### 平面地图的测绘功能，可在测绘图片上设定比例尺、测距，计算面积、规划区域等，并将最终测绘结果下载为png图片(原图)。

### 使用
[leaflet](https://github.com/Leaflet/Leaflet "leaflet")(v0.7.7) <br/>
[leaflet.draw](https://github.com/Leaflet/Leaflet.draw "leaflet.draw")(v0.4.9) <br/>
[leaflet.label](https://github.com/Leaflet/Leaflet.label "leaflet.label") <br/>
[sweetalert2](https://github.com/limonte/sweetalert2 "sweetalert2") <br/>
[pinyinjs](https://github.com/sxei/pinyinjs "pinyinjs") <br/>
[jquery](https://github.com/jquery/jquery "jquery")(v3.1.1)<br>

基于[leaflet-image-hotspots](https://github.com/ZanwingMak/leaflet-image-hotspots)  实现热点图像区域编辑功能，能够在图片上绘制、编辑、删除图形区域以及文字说明。

### Github
[https://github.com/xiaomai0830/leaflet-image-hotspots](https://github.com/xiaomai0830/leaflet-image-hotspots "https://github.com/xiaomai0830/leaflet-image-hotspots")<br/>
[https://github.com/ZanwingMak/leaflet-image-hotspots](https://github.com/ZanwingMak/leaflet-image-hotspots)

##设置比例尺
1.点击比例尺时激活地图点击事件
```javascript
function createScale(e) {
    e.stopPropagation();
    let sLen = scale_geometry.length;
    if(sLen>0){
        for (let i = 0 ; i<sLen; i++){
            map.removeLayer(scale_geometry[i]);
        }
    }
    scale_line = L.polyline(scaleArr,{color:'#000000'});
    map.on("click",addScale);
}
```
2.添加点，存入scale_geometry数组以便在重设比例尺时移除地图上的比例尺图形。
```javascript
function addScale(e){
    scaleArr.push([e.latlng.lat, e.latlng.lng]);
    scale_line.addLatLng(e.latlng);
    map.addLayer(scale_line);
    const node=new L.circleMarker(e.latlng , { color: '#363333', fillColor: '#363030', fillOpacity: 1 ,radius:5 });
    map.addLayer(node);
    scale_geometry.push(node);
    map.on('mousemove', scale_lineMove);//双击地图
    if (scaleArr.length == 2){
        scale_end();
    }
}
```
3.鼠标移动跟随线段
```javascript
function scale_lineMove(e){
    if (scaleArr.length > 0) {
        let ls = [scaleArr[scaleArr.length - 1], [e.latlng.lat, e.latlng.lng]]
        tempLine.setLatLngs(ls);
        map.addLayer(tempLine);
    }
}
```
4.结束比例尺绘制并弹出设置框
```javascript
function scale_end(e){
    scaleArr = [];
    scale_geometry.push(scale_line);
    map.removeLayer(tempLine);
    map.off('mousemove');
    map.off("click",addScale);
    //弹出设置框
    swal({
        title: '请输入正整数（单位为米）',
        input: 'text',
        showCancelButton: true,
        inputValidator: function (value) {
            return new Promise(function (resolve, reject) {
                let r = /^\+?[1-9][0-9]*$/;
                if (r.test(value)) {
                    resolve();
                    scale=parseInt(value);
                    scale_length = computeLength(scale_line.getLatLngs());
                } else {
                    reject('无效输入，请输入正整数！');
                }
            })
        }
    }).then(function (result) {
        scale_line.bindLabel('比例尺');
        scale_line.setText(scale+"m");

        $.each(drawnItems._layers,function (i,layer) {
            resetText(layer);
        })

        swal({
            type: 'success',
            html: '比例尺设置成功。'
        })
    }, function (dismiss) {
        for (let i = 0 ; i<scale_geometry.length; i++){
            map.removeLayer(scale_geometry[i]);
        }
    });
}
```

###重新设置图形上的文本(显示面积于图形上)
```javascript
function resetText(layer){
    let id = layer._leaflet_id;
    let type;
    let latlng = layer.getLatLngs();
    if (scale_length == 0){
        return;
    }
    $.each(pointAttr,function (i,item) {
        if (item.leaflet_id == id){
            type = item.type;
            return false;
        }
    });
    if (type == "polyline"){
        let distance = (computeLength(latlng) / scale_length * scale).toFixed(2);
        layer.setText(distance+"m");
    }else{
        let area = (computePolygonArea(latlng)/scale_length/scale_length*scale*scale).toFixed(2);
        layer.setText(area +"m²");
    }
}
```

## 面积及路径计算
leaflet自带的 L.GeometryUtil.geodesicArea() 计算面积方法，和 L.latLng().distanceTo() 计算两点间的距离的算法不适用于该平面测绘.
另找了一个计算面积的方法[计算任意多边形面积](https://blog.csdn.net/mailzst1/article/details/89554199) <br/>
1.计算面积
```javascript
    function computePolygonArea(lnglats) {
    　　let length = lnglats.length;
    　　let s = lnglats[0].lng * (lnglats[length - 1].lat - lnglats[1].lat);
    　　for (let i =  1;i < length; i++){
    　　　　s += lnglats[i].lng * (lnglats[i-1].lat - lnglats[(i+1) % length].lat);
    　　}
    　　return Math.abs(s/2);
    }
```
2.计算长度
```javascript
   function computeLength(line){
   　　let dis = 0;
   　　for (let i = 0; i <line.length - 1;i++){
   　　　　let start = line[i];
   　　　　let end = line[i+1];
   　　　　let dx = start.lng - end.lng;
   　　　　let dy = start.lat -end.lat;
   　　　　dis += Math.sqrt(dx*dx + dy*dy);
   　　}
   　　return dis;
   }
```

