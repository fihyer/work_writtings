# img_processor.py

```python
# -*- coding: utf-8 -*-
from pathlib import Path
import re

import cv2 as cv
import numpy as np
from utils import LogLevel, Logger
import baidu


img_logger = Logger(__file__, log_level=LogLevel.NORMAL)

ai_ocr = baidu.BaiduAI()


def template_matching(data_full_path, template_full_path):
    final_result = {}
    validator = {}
    for target_path in tuple(Path(data_full_path).glob('*.bmp')):
        target = str(target_path)
        target_img = cv.imread(target, 0)
        # for each image saved in the first stage
        # match it with template image
        for template_path in tuple(Path(template_full_path).glob('*.bmp')):
            template = str(template_path)
            template_number = template.split('.')[1]
            template_img = cv.imread(template, 0)
            w, h = template_img.shape[::-1]

            img = target_img.copy()
            # If using 'TM_CCOEFF_NORMED' method the top left coordination is max_loc
            method = eval('cv.TM_CCOEFF_NORMED')
            result = cv.matchTemplate(img, template_img, method)
            # unpacking the template matching result
            _, confidence, _, max_loc = cv.minMaxLoc(result)
            # for matching confidence greater than 0.8,
            # save useful infos: template image number, top left coordination,
            # and template image size into dictionary with screenshots image as key
            if confidence > 0.8:
                final_result[target] = [template_number, max_loc, w, h]
            else:
                validator[target] = [template_number]
    # Cross-validating result dictionary to lock 3 different image
    for key, _ in validator.items():
        if not final_result.get(key):
            final_result[key] = ['1']
    return final_result


def indices(temp_dict, value):
    """
    :type temp_dict: dictionary.
    :type value: number like string.
    """
    images = []
    img_path_list = list(temp_dict.keys())
    img_meta_list = [item[0] for item in list(temp_dict.values())]
    for idx, val in enumerate(img_meta_list):
        if val == value:
            images.append(img_path_list[idx])
    return images


def first_processor(images):
    temp_file_list = []
    img_logger.write_to_log("-" * 70, log_level=LogLevel.NORMAL)
    for image in images:
        img_logger.write_to_log(f"开始处理用户信息图片： {image}", log_level=LogLevel.NORMAL)
        img = cv.imread(image)
        w, h = img.shape[:-1]
        # Manually define the ocr detecting area:
        # from (0,0) to (1/3 width and height) of the full image
        detect_area = img[0:int(w / 3), 0:int(h / 3)]
        gray_image = cv.cvtColor(detect_area, cv.COLOR_BGR2GRAY)
        # (83, 242) is best parameter to suite the first image
        (thresh, black_and_white) = cv.threshold(gray_image, 83, 242,
                                                 cv.THRESH_BINARY)

        gray = np.float32(black_and_white)

        # set Harris edge detector parameters:
        # block size: 2
        # Extended Sobel kernel(1, 3, 5, or 7): 3
        # free parameter: 0.04
        dst = cv.cornerHarris(gray, 2, 3, 0.04)
        dst = cv.dilate(dst, None)
        ret, dst = cv.threshold(dst, 0.01 * dst.max(), 255, 0)
        dst = np.uint8(dst)

        ret, labels, stats, centroids = cv.connectedComponentsWithStats(dst)

        criteria = (cv.TERM_CRITERIA_EPS + cv.TERM_CRITERIA_MAX_ITER, 100, 0.01)
        corners = cv.cornerSubPix(black_and_white, np.float32(centroids), (5, 5),
                                  (-1, -1), criteria)

        result = np.hstack((centroids, corners))
        result = np.int0(result)

        img[result[:, 1], result[:, 0]] = [0, 0, 255]
        img[result[:, 3], result[:, 2]] = [0, 255, 0]

        path_parts = image.split('.')
        saving_path = path_parts[0] + '-first.' + path_parts[1]
        cv.imwrite(saving_path, black_and_white)
        temp_file_list.append(saving_path)
        img_logger.write_to_log(f"用户信息中间文件保存至： {saving_path}", log_level=LogLevel.NORMAL)
    return temp_file_list


def second_processor(images, temp_match_dict):
    temp_file_list = []
    img_logger.write_to_log("-" * 70, log_level=LogLevel.NORMAL)
    for img_file in images:
        img_logger.write_to_log(f"正在处理并分析器官图片: {img_file}")
        color_img = cv.imread(img_file)
        color_img_w, color_img_h = color_img.shape[:-1]

        number, top_left, w, h = temp_match_dict[img_file]
        bottom_right = (top_left[0] - 2 + int(w * 28.5),
                        top_left[1] - 2 + int(h * 12.5))
        cropped = color_img[top_left[1]:bottom_right[1], top_left[0]:bottom_right[0]]

        _img = color_img.copy()
        h_start = abs(color_img_w - 60)
        w_end = int(color_img_h/3)
        organ_text_area = _img[h_start:color_img_w, 0:w_end]

        path_parts = img_file.split('.')

        organ_text_img_path = path_parts[0] + '-organ_text.' + path_parts[1]
        cv.imwrite(organ_text_img_path, organ_text_area)
        organ_name_result = ai_ocr.img_to_text(img_file=organ_text_img_path)
        numeric_const_pattern = r"""
        (?:
            (?: \d* \. \d+)
            |
            (?: \d+ \.?)
        )
        """
        numbers = re.compile(numeric_const_pattern, re.VERBOSE)
        if organ_name_result.get('error_code'):
            ocr_error_code = organ_name_result['error_code']
            temp_file_list.append(ocr_error_code)
        else:
            organ_name_str = organ_name_result['words_result'][0]['words']
            organ_code = numbers.findall(organ_name_str)[0]
            img_logger.write_to_log(f"{organ_name_str} 器官图片编号为 {organ_code}", log_level=LogLevel.NORMAL)
            # For the sick of file uploading requirement, change image extension to png
            saving_path = path_parts[0] + '-' + organ_code + '.png'

            cv.imwrite(saving_path, cropped)
            temp_file_list.append(saving_path)
            img_logger.write_to_log(f"器官图片中间文件保存至： {saving_path}", log_level=LogLevel.NORMAL)
    return temp_file_list


def third_processor(images, temp_match_dict):
    temp_file_list = []
    img_logger.write_to_log("-" * 70, log_level=LogLevel.NORMAL)
    for img_file in images:
        img_logger.write_to_log(f"开始处理结果图片： {img_file}", log_level=LogLevel.NORMAL)
        img = cv.imread(img_file)

        _, top_left, w, h = temp_match_dict[img_file]
        bottom_right = (top_left[0] + int(w * 17.5), top_left[1] + int(h * 9.5))
        cropped = img[top_left[1]:bottom_right[1], top_left[0]:bottom_right[0]]

        path_parts = img_file.split('.')
        saving_path = path_parts[0] + '-third.' + path_parts[1]

        cv.imwrite(saving_path, cropped)
        temp_file_list.append(saving_path)
        img_logger.write_to_log(f"结果图片中间文件保存至： {saving_path}", log_level=LogLevel.NORMAL)
    return temp_file_list


def user_data_extractor(img_file):
    img_logger.write_to_log("-" * 70, log_level=LogLevel.NORMAL)
    img_logger.write_to_log(f"开始分析并清洗用户信息： {img_file}", log_level=LogLevel.NORMAL)
    user_info = {}
    text_result = ai_ocr.double_checker(img_file)
    img_logger.write_to_log("double checking done!", log_level=LogLevel.NORMAL)

    # Remove redundant characters
    alphabet_pattern = r"[^a-z|A-Z]"
    alphabet_regex = re.compile(alphabet_pattern, re.VERBOSE)

    for key, words_list in text_result.items():
        user_info[key] = []
        if words_list:
            for word in words_list['words_result']:
                alphabet = alphabet_regex.findall(word['words'])
                validated_infos = ''.join(alphabet)
                user_info[key].append(validated_infos)
        else:
            user_info[key] = None
    img_logger.write_to_log(f"首次清洗结果： {user_info}", log_level=LogLevel.NORMAL)

    final_infos = {}
    for ocr_method, infos in user_info.items():
        if infos:
            name_pos = infos.index('姓名')
            age_pos = infos.index('年龄')
            gender_pos = infos.index('性别')
            address_pos = infos.index('地址')
            # user name always come in the first place after ‘姓名'
            name = infos[name_pos + 1]
            # if '年龄' and ’性别' has the same index -> age info not recognized
            if age_pos + 1 == gender_pos:
                age = None
            else:
                age = infos[age_pos + 1]
            gender = infos[gender_pos + 1]
            if address_pos + 1 == len(infos):
                address = None
            else:
                address = infos[address_pos + 1]

            final_infos[ocr_method] = [name, age, gender, address]
        else:
            final_infos[ocr_method] = None

    img_logger.write_to_log(f"第二次清洗结果： {final_infos}", log_level=LogLevel.NORMAL)

    ret = []
    if final_infos['accurate']:
        if None in final_infos['basic']:
            none_val_index = final_infos['basic'].index(None)
            final_infos['basic'][none_val_index] = final_infos['accurate'][none_val_index]

        if None in final_infos['accurate']:
            none_val_index = final_infos['accurate'].index(None)
            final_infos['accurate'][none_val_index] = final_infos['basic'][none_val_index]
        ret = final_infos['accurate']
    else:
        ret = final_infos['basic']

    img_logger.write_to_log(f"最终清洗结果： {ret}", log_level=LogLevel.NORMAL)

    return ret


def analysis_result_extractor(img_file):
    img_logger.write_to_log("-" * 70, log_level=LogLevel.NORMAL)
    img_logger.write_to_log(f"开始分析结果图片： {img_file}", log_level=LogLevel.NORMAL)
    result = []
    analysis = ai_ocr.img_to_text(img_file)

    numeric_const_pattern = r"""
    (?:
        (?: \d* \. \d+)
        |
        (?: \d+ \.?)
    )
    """
    alphabet_const_pattern = r"[^a-z|A-Z]"
    left_parenthesis_pattern = r"(.*?)(?=\()"

    numbers = re.compile(numeric_const_pattern, re.VERBOSE)
    alphabet = re.compile(alphabet_const_pattern, re.VERBOSE)
    l_parenthesis = re.compile(left_parenthesis_pattern, re.VERBOSE)

    img_logger.write_to_log(f"基础文字识别结果：\n", log_level=LogLevel.NORMAL)
    for v_idx, vals in enumerate(analysis['words_result']):
        img_logger.write_to_log(f"{v_idx}： {vals}", log_level=LogLevel.NORMAL)
    for item in analysis['words_result'][2:7]:
        # Separating spectrum value and disease name
        value_str_list = numbers.findall(item['words'])
        img_logger.write_to_log(f"文字中包含数字： {value_str_list}", log_level=LogLevel.NORMAL)

        value_str = '0555'
        if value_str_list:
            value_str = value_str_list[0]
            start_pos = len(value_str)
            img_logger.write_to_log(f"数字长度： {start_pos}", log_level=LogLevel.NORMAL)
            if len(value_str_list) > 1:
                img_logger.write_to_log("文字中有两个以上类似数字串", log_level=LogLevel.NORMAL)
                disease_name = item['words'][start_pos:]
                img_logger.write_to_log(f"剔除疾病前的数字串后： {disease_name}", log_level=LogLevel.NORMAL)
                end_nums = numbers.findall(disease_name)
                if end_nums:
                    end_pos = disease_name.find(end_nums[0])
                    disease_name = disease_name[:end_pos]
                    img_logger.write_to_log(f"剔除疾病后的数字串后： {disease_name}", log_level=LogLevel.NORMAL)
            elif item['words'].index(value_str) + start_pos < len(item['words']):
                disease_name = item['words'][start_pos:]
            else:
                value_str = '0555'
                disease_name = item['words'][:start_pos]
        else:
            disease_name = item['words']

        # remove parenthesis if has any at the end
        redundant_parenthesis = l_parenthesis.findall(disease_name)
        if redundant_parenthesis:
            disease_name = redundant_parenthesis[0]

        # remove leading alphabets if any table line miss recognized to alphabet
        alphabets = alphabet.findall(disease_name)
        if alphabets:
            disease_name = ''.join(alphabets)

        if len(value_str) > 6:
            value_str = value_str[2:]
        elif len(value_str) <= 2:
            value_str = '0555'
        else:
            value_str = value_str[1:]

        if value_str.find('.') == -1:
            value_str = value_str[:1] + '.' + value_str[1:]

        spectrum_value = float(value_str)
        if 0 < spectrum_value <= 0.425:
            disease_level = 1
        elif 0.425 < spectrum_value <= 1.0:
            disease_level = 2
        else:
            disease_level = 0

        if disease_level:
            result.append(
                [disease_name, disease_level, spectrum_value])
    img_logger.write_to_log(f"结果图片: {img_file} 的分析结果为: {result}", log_level=LogLevel.NORMAL)

    return result
```

