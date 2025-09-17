---
layout: page
title: Activity_강제_실행
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

```python
import xml.etree.ElementTree as ET
import subprocess
import sys

# 20240829 https://7-3-7.tistory.com/

def extract_package_name(manifest_file):
    # AndroidManifest.xml 파일 파싱
    tree = ET.parse(manifest_file)
    root = tree.getroot()

    # 패키지 이름 추출
    return root.get('package')

def extract_activities(manifest_file):
    # AndroidManifest.xml 파일 파싱
    tree = ET.parse(manifest_file)
    root = tree.getroot()

    # 네임스페이스 정의
    android_namespace = 'http://schemas.android.com/apk/res/android'
    
    # 모든 Activity 이름 추출
    activities = []
    for activity in root.findall(".//activity"):
        activity_name = activity.get(f'{{{android_namespace}}}name')
        if activity_name:
            activities.append(activity_name)

    return activities

def generate_am_start_commands(package_name, activities):
    commands = []
    for activity in activities:
        # adb shell 명령어 생성
        command = f"adb shell am start -n {package_name}/{activity}"
        commands.append(command)
    return commands

def check_security(commands):
    results = {'취약': [], '양호': [], '오류': []}
    total_commands = len(commands)

    for idx, command in enumerate(commands):
        try:
            # 명령어 실행
            result = subprocess.run(command, shell=True, stderr=subprocess.PIPE, stdout=subprocess.PIPE, text=True)

            # stderr가 비어있지 않다면 오류가 발생한 것
            if result.returncode != 0 or result.stderr:
                results['양호'].append(command)
            else:
                results['취약'].append(command)
        
        except Exception as e:
            # 명령어 실행 중 예외 발생 시 오류로 처리
            results['오류'].append(f"{command} -> 오류 발생: {str(e)}")

        # 진행 상황 출력
        percent_complete = (idx + 1) / total_commands * 100
        sys.stdout.write(f"\r진행 상황: {percent_complete:.2f}% 완료")
        sys.stdout.flush()
    
    print()  # 진행 상황 표시 후 새로운 줄로 이동
    return results

# 사용 예시
manifest_file = 'AndroidManifest.xml'

# 패키지 이름 추출
package_name = extract_package_name(manifest_file)

# Activity 목록 추출
activities = extract_activities(manifest_file)

# adb shell am start 명령어 생성
commands = generate_am_start_commands(package_name, activities)

# 보안 체크 실행 및 결과 출력
results = check_security(commands)

print("\n취약한 명령어:")
for command in results['취약']:
    print(command)

# 양호한 명령어를 볼지 여부 묻기
view_safe = input("\n양호한 명령어를 보시겠습니까? (Y/N): ").strip().upper()

if view_safe == 'Y':
    print("\n양호한 명령어:")
    for command in results['양호']:
        print(command)

print("\n오류 발생한 명령어:")
for command in results['오류']:
    print(command)

print(f"\nTotal number of activities: {len(commands)}")

```


Activity_check.py 파일과 대상 apk 파일의 AndroidManifest.xml 파일을 한 폴더에 넣고 실행 시킨다.
apk 파일 압축 풀고 AndroidManifest.xml 추출하면 내용이 다 깨지기 때문에 jadx에서 Export로 추출한다.

