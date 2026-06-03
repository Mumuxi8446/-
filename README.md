#include <iostream>
#include <vector>
#include <cmath>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    // 读取图片
    Mat src = imread("D:/project/111/rm/zjb.png");
    if (src.empty()) {
        cout << "load fail" << endl;
        return -1;
    }

    Mat result = src.clone();
    Mat gray;
    cvtColor(src, gray, COLOR_BGR2GRAY);

    // 直接二值化，不做边缘检测
    Mat binary;
    threshold(gray, binary, 180, 255, THRESH_BINARY);

    // 只使用闭运算连接断开的区域，不使用开运算
   // 形态学操作 - 连接断开的区域
    Mat kernel = getStructuringElement(MORPH_RECT, Size(5, 5));
    dilate(binary, binary, kernel);  // 膨胀
    dilate(binary, binary, kernel);  // 再膨胀一次
    morphologyEx(binary, binary, MORPH_CLOSE, kernel);


    // 寻找轮廓
    vector<vector<Point>> contours;
    findContours(binary, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);

    cout << "Found " << contours.size() << " contours" << endl;

    // 存储灯条
    vector<RotatedRect> lightBars;

    // 筛选灯条
    for (const auto& contour : contours) {
        RotatedRect rect = minAreaRect(contour);
        Size2f size = rect.size;

        float height = max(size.width, size.height);
        float width = min(size.width, size.height);
        float ratio = height / width;
        float angle = rect.angle;

        if (size.width > size.height) {
            angle = 90 - angle;
        }

        float area = contourArea(contour);

        // 打印每个轮廓的信息，查看左边灯条是否被筛掉
        cout << "Contour: area=" << area << ", aspect_ratio=" << ratio
            << ", angle=" << angle << ", width=" << width << endl;

        // 放宽所有条件
        if (ratio > 1.5 && ratio < 15.0 &&
            area > 20 && area < 10000 &&
            abs(angle) < 80 &&
            width < 80) {

            lightBars.push_back(rect);

            Point2f vertices[4];
            rect.points(vertices);
            for (int k = 0; k < 4; k++) {
                line(result, vertices[k], vertices[(k + 1) % 4], Scalar(0, 255, 0), 2);
            }
            circle(result, rect.center, 5, Scalar(0, 0, 255), -1);

            cout << "-> Identified as light bar!" << endl;
        }
    }

    cout << "\nDetected " << lightBars.size() << " light bars" << endl;

    // 匹配装甲板
    int armorCount = 0;
    for (size_t i = 0; i < lightBars.size(); i++) {
        for (size_t j = i + 1; j < lightBars.size(); j++) {
            Point2f center1 = lightBars[i].center;
            Point2f center2 = lightBars[j].center;

            float distance = norm(center1 - center2);
            float yDiff = abs(center1.y - center2.y);

            if (distance > 30 && distance < 500 && yDiff < 100) {
                armorCount++;

                Point2f armorCenter = (center1 + center2) / 2;
                circle(result, armorCenter, 8, Scalar(255, 0, 0), -1);
                
                putText(result, to_string(armorCount),
                    armorCenter - Point2f(10, 15),
                    FONT_HERSHEY_SIMPLEX, 0.8, Scalar(0, 255, 255), 2);

                cout << "Armor plate " << armorCount << " center: (" << armorCenter.x << ", " << armorCenter.y << ")" << endl;
            }
        }
    }

    cout << "Found " << armorCount << " armor plates" << endl;

    imshow("Detected", result);
    imwrite("armor_result.jpg", result);

    waitKey(0);
    return 0;
}
