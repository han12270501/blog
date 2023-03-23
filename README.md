// 创建地图
var map = new BMap.Map("container");

// 创建线路
var path = [
    new BMap.Point(116.399, 39.910),
    new BMap.Point(116.417, 39.920),
    new BMap.Point(116.430, 39.900)
];
var polyline = new BMap.Polyline(path, {strokeColor:"blue", strokeWeight:6, strokeOpacity:0.5});

// 添加线路和 Marker 到地图
map.addOverlay(polyline);
var marker = new BMap.Marker(path[1]);
marker.setZIndex(9999);
marker.enableDragging();
map.addOverlay(marker);

// 监听 Marker 拖拽事件
marker.addEventListener("dragging", function(e) {
    // 获取 Marker 的当前位置
    var point = e.point;

    // 计算 Marker 到线路的最近点
    var nearestPoint = getNearestPointOnPolyline(point, polyline.getPath());

    // 更新 Marker 的位置
    marker.setPosition(nearestPoint);
});

// 计算点到折线段的垂足
function getNearestPointOnPolyline(point, polylinePath) {
    var minDist = Number.MAX_VALUE;
    var nearestPoint = null;
    for (var i = 0; i < polylinePath.length - 1; i++) {
        var dist = BMapLib.GeoUtils.getDistanceToSegment(point, polylinePath[i], polylinePath[i + 1]);
        if (dist < minDist) {
            minDist = dist;
            nearestPoint = BMapLib.GeoUtils.getPointOfSegmentDistance(point, polylinePath[i], polylinePath[i + 1]);
        }
    }
    return nearestPoint;
}
