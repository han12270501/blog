// 创建地图实例
var map = new BMapGL.Map("map");

// 创建折线对象
var polyline = new BMapGL.Polyline([
    new BMapGL.Point(116.399, 39.910),
    new BMapGL.Point(116.405, 39.920),
    new BMapGL.Point(116.425, 39.900)
], {strokeColor: "blue", strokeWeight: 6, strokeOpacity: 0.5});

// 将折线添加到地图中
map.addOverlay(polyline);

// 创建可拖拽的标记点，并添加到地图中
var marker = new BMapGL.Marker(new BMapGL.Point(116.405, 39.920), {enableDragging: true});
map.addOverlay(marker);

// 绑定标记点拖拽事件
marker.addEventListener("dragging", function(e) {
    var point = e.point;
    
    // 判断点是否在折线内
    if (BMapGLLib.GeoUtils.isPointInPoly(point, polyline)) {
        marker.setPosition(point);  // 设置标记点位置
    } else {
        // 找出点到折线段的垂足，并将标记点移动到该位置
        var foot = getFoot(point, polyline);
        marker.setPosition(foot);
    }
});

/**
 * 获取点到折线段的垂足
 * @param point 点的坐标
 * @param polyline 折线对象
 * @returns 垂足的坐标
 */
function getFoot(point, polyline) {
    var path = polyline.getPath();  // 获取折线路径
    var minDist = Infinity;  // 初始化最小距离为无限大
    var foot;  // 垂足坐标
    
    // 遍历折线每个线段，求出点到线段的垂足
    for (var i = 0; i < path.length - 1; i++) {
        var start = path[i];
        var end = path[i + 1];
        var dist = getDistanceToSegment(start, end, point);  // 计算点到线段距离
        
        // 如果距离小于当前最小距离，则更新最小距离和垂足坐标
        if (dist < minDist) {
            minDist = dist;
            foot = getFootFromPointToSegment(start, end, point);
        }
    }
    
    return foot;
}

/**
 * 获取点到线段的距离
 * @param start 线段起点
 * @param end 线段终点
 * @param point 点的坐标
 * @returns 点到线段的距离
 */
function getDistanceToSegment(start, end, point) {
    var A = point.lng - start.lng;
    var B = point.lat - start.lat;
    var C = end.lng - start.lng;
    var D = end.lat - start.lat;

    var dot = A * C + B * D;
    var len_sq = C * C + D * D;
    var param = dot / len_sq;

    var xx, yy;

    if (param < 0 || (start.lng == end.lng && start.lat == end.lat)) {
        xx = start.lng;
        yy = start.lat;
    } else if (param > 1) {
        xx = end.lng;
        yy = end.lat;
    } else {
        xx = start.lng + param * C;
        yy = start.lat + param * D;
    }

    var dx = point.lng - xx;
    var dy = point.lat - yy;
    return Math.sqrt(dx * dx + dy * dy);
}

/**
 * 获取点到线段的垂足
 * @param start 线段起点
 * @param end 线段终点
 * @param point 点的坐标
 * @returns 垂足的坐标
 */
function getFootFromPointToSegment(start, end, point) {
    var A = point.lng - start.lng;
    var B = point.lat - start.lat;
    var C = end.lng - start.lng;
    var D = end.lat - start.lat;

    var dot = A * C + B * D;
    var len_sq = C * C + D * D;
    var param = dot / len_sq;

    var xx, yy;

    if (param < 0 || (start.lng == end.lng && start.lat == end.lat)) {
        xx = start.lng;
        yy = start.lat;
    } else if (param > 1) {
        xx = end.lng;
        yy = end.lat;
    } else {
        xx = start.lng + param * C;
        yy = start.lat + param * D;
    }

    return new BMapGL.Point(xx, yy);
}
