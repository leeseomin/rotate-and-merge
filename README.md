# 🟡  rotate-and-merge in osx terminal 

🟩 Perform rotate and merge operations on images within a specific folder.

🟧 The images must have equal width and height.

🟡 ``` sh ``` : script entry in terminal 


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



1:1 크롭하여 실행하기

```

#!/bin/bash

# 임시 디렉토리 생성
mkdir -p temp_images

# 현재 폴더 내의 모든 이미지 파일에 대해 반복
for original in *.{jpg,jpeg,png,gif,bmp,tiff,webp}; do
    # 파일이 실제로 존재하는지 확인
    [[ ! -f "$original" ]] && continue
    
    # 파일 확장자 및 이름 추출
    filename=$(basename "$original")
    extension="${filename##*.}"
    filename="${filename%.*}"
    
    echo "Processing: $original"
    
    # 원본 이미지의 크기 정보 가져오기
    dimensions=$(identify -format "%wx%h" "$original")
    width=$(echo $dimensions | cut -d'x' -f1)
    height=$(echo $dimensions | cut -d'x' -f2)
    
    # 정사각형 크롭을 위한 최소 크기 계산
    min_size=$((width < height ? width : height))
    
    # 1:1 비율로 중앙 크롭
    convert "$original" -gravity center -crop "${min_size}x${min_size}+0+0" +repage "temp_images/base.png"
    
    # 다양한 변형 효과로 16개 버전 생성 (4x4 그리드)
    effects=(
        ""                                          # 원본
        "-rotate 90"                               # 90도 회전
        "-rotate 180"                              # 180도 회전  
        "-rotate 270"                              # 270도 회전
        "-flop"                                    # 좌우 반전
        "-flip"                                    # 상하 반전
        "-flop -flip"                              # 좌우+상하 반전
        "-rotate 45 -crop ${min_size}x${min_size}+0+0 +repage"  # 45도 회전 후 크롭
        "-modulate 120,150,100"                    # 밝기/채도 조정
        "-negate"                                  # 색상 반전
        "-colorize 30,0,30"                        # 보라색 틴트
        "-colorize 0,30,30"                        # 청록색 틴트
        "-blur 0x2"                                # 블러 효과
        "-sharpen 0x1"                             # 샤픈 효과
        "-emboss 2"                                # 엠보스 효과
        "-solarize 50%"                            # 솔라라이즈 효과
    )
    
    # 각 효과 적용하여 이미지 생성
    for i in "${!effects[@]}"; do
        if [[ -n "${effects[$i]}" ]]; then
            convert "temp_images/base.png" ${effects[$i]} "temp_images/$(printf "%02d" $i).png"
        else
            cp "temp_images/base.png" "temp_images/$(printf "%02d" $i).png"
        fi
    done
    
    # 4x4 그리드로 몽타주 생성 (간격 최소화)
    montage temp_images/[0-9][0-9].png \
        -tile 4x4 \
        -geometry +2+2 \
        -background black \
        "${filename}_collage_4x4.${extension}"
    
    # 6x6 그리드를 위한 추가 변형 (36개 버전)
    echo "Creating extended collage for $original"
    
    # 추가 효과들
    additional_effects=(
        "-rotate 15"
        "-rotate 30" 
        "-rotate 60"
        "-rotate 120"
        "-rotate 135"
        "-rotate 150"
        "-rotate 210"
        "-rotate 225"
        "-rotate 240"
        "-rotate 300"
        "-rotate 315"
        "-rotate 330"
        "-scale 80% -gravity center -extent ${min_size}x${min_size}"
        "-scale 120% -gravity center -crop ${min_size}x${min_size}+0+0 +repage"
        "-swirl 30"
        "-swirl -30"
        "-wave 10x50"
        "-implode 0.3"
        "-implode -0.3"
        "-charcoal 2"
    )
    
    # 추가 이미지들 생성
    for i in "${!additional_effects[@]}"; do
        idx=$((i + 16))
        convert "temp_images/base.png" ${additional_effects[$i]} "temp_images/$(printf "%02d" $idx).png"
    done
    
    # 6x6 그리드로 몽타주 생성
    montage temp_images/[0-9][0-9].png \
        -tile 6x6 \
        -geometry +1+1 \
        -background black \
        "${filename}_collage_6x6.${extension}"
    
    # 원형 마스크 버전도 생성
    echo "Creating circular masked version for $original"
    
    # 원형 마스크 생성
    convert -size "${min_size}x${min_size}" xc:black \
        -fill white -draw "circle $((min_size/2)),$((min_size/2)) $((min_size/2)),0" \
        temp_images/mask.png
    
    # 원형 크롭된 버전들 생성
    for i in {0..15}; do
        convert "temp_images/$(printf "%02d" $i).png" temp_images/mask.png -alpha off -compose copy_opacity -composite "temp_images/circle_$(printf "%02d" $i).png"
    done
    
    # 원형 버전 4x4 몽타주
    montage temp_images/circle_[0-9][0-9].png \
        -tile 4x4 \
        -geometry +5+5 \
        -background white \
        "${filename}_circle_collage.${extension}"
    
    # 임시 파일들 정리
    rm temp_images/*.png
    
    echo "Completed: ${filename}_collage_4x4.${extension}, ${filename}_collage_6x6.${extension}, ${filename}_circle_collage.${extension}"
done

# 임시 디렉토리 삭제
rmdir temp_images

echo "All collages created successfully!"
```





### 5 x 5 rotate and merge 



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
    for i in {1..24}; do
        angle=$((90 * i))
        convert "temp_images/0.png" -rotate $angle "temp_images/$i.png"
    done

    # 모든 이미지를 병합하여 최종 파일 저장
    montage temp_images/*.png -tile 5x5 -geometry +0+0 "${filename}_merged.${extension}"

    # 임시 이미지 파일 삭제
    rm temp_images/*.png
done

# 임시 디렉토리 삭제
rmdir temp_images
```



### 7 x 7 rotate and merge 



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
    for i in {1..48}; do
        angle=$((90 * i))
        convert "temp_images/0.png" -rotate $angle "temp_images/$i.png"
    done

    # 모든 이미지를 병합하여 최종 파일 저장
    montage temp_images/*.png -tile 7x7 -geometry +0+0 "${filename}_merged.${extension}"

    # 임시 이미지 파일 삭제
    rm temp_images/*.png
done

# 임시 디렉토리 삭제
rmdir temp_images
```



### 9 x 9 rotate and merge 



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
    for i in {1..80}; do
        angle=$((90 * i))
        convert "temp_images/0.png" -rotate $angle "temp_images/$i.png"
    done

    # 모든 이미지를 병합하여 최종 파일 저장
    montage temp_images/*.png -tile 9x9 -geometry +0+0 "${filename}_merged.${extension}"

    # 임시 이미지 파일 삭제
    rm temp_images/*.png
done

# 임시 디렉토리 삭제
rmdir temp_images
```



