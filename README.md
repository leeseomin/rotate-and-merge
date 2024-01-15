# 🟡  rotate-and-merge in terminal 

rotate and merge operations on images



### 3 x 3 rotate and merge 

```

#!/bin/bash

# 임시 디렉토리 생성
mkdir temp_images

# 현재 폴더 내의 모든 이미지 파일에 대해 반복
for original in *.*; do
    # 파일 확장자 및 이름 추출
    filename=$(basename "$original")
    extension="${filename##*.}"
    filename="${filename%.*}"

    # 원본 이미지를 temp_images 디렉토리에 복사
    cp "$original" "temp_images/0.png"

    # 이미지를 복제하고 90도씩 회전
    for i in {1..8}; do
        angle=$((90 * i))
        convert "temp_images/0.png" -rotate $angle "temp_images/$i.png"
    done

    # 모든 이미지를 병합하여 최종 파일 저장
    montage temp_images/*.png -tile 3x3 -geometry +0+0 "${filename}_merged.${extension}"

    # 임시 이미지 파일 삭제
    rm temp_images/*.png
done

# 임시 디렉토리 삭제
rmdir temp_images


```
