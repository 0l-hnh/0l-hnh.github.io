---
layout: single
title:  "[ASR] ESPnet yesno 예제 정상 실행 LOG"
date:   2023-01-21 11:10:00 +0900

categories:
  - asr
tags: [asr, espnet, docker]

author_profile: true
sidebar:
  nav: "docs"

toc: true
toc_label: "목록"
toc_icon: "bars"
toc_sticky: true

sitemap:
  changefreq: daily
  priority : 1.0
---

## 설명  
본 문서는 nvidia-docker로 ESPnet2 훈련 예제 중 yesno 예제 실행 시 출력되는 LOG 기록용으로 작성되었다.  

## 상세  
### LOG 전문    
```bash
$ ./run.sh --docker-gpu 0 --is-egs2 --docker-egs yesno/asr1 --ngpu 1                                                                                                                Using image espnet/espnet:gpu-latest-user-user.
Executing application in Docker
NV_GPU='0' nvidia-docker run -i --rm --name espnet_gpu0_20221125T1819 -v /data/espnet/egs:/espnet/egs -v /data/espnet/espnet:/espnet/espnet -v /data/espnet/test:/espnet/test -v /data/espnet/utils:/espnet/ls -v /data/espnet/egs2:/espnet/egs2 -v /data/espnet/espnet2:/espnet/espnet2 -v /dev/shm:/dev/shm espnet/espnet:gpu-latest-user-user /bin/bash -c 'cd /espnet/egs2/yesno/asr1; ./run.sh --ngpu 1'
2022-11-25T09:19:25 (asr.sh:255:main) ./asr.sh --lang en --audio_format wav --feats_type raw --token_type char --use_lm false --asr_config conf/train_asr.yaml --inference_config conf/decode.yaml --train_set train_nodev --valid_set train_dev --test_sets train_dev test_yesno --lm_train_text data/train_nodev/text --ngpu 1
2022-11-25T09:19:25 (asr.sh:456:main) Stage 1: Data preparation for data/train_nodev, data/train_dev, etc.
2022-11-25T09:19:25 (data.sh:16:main) local/data.sh
2022-11-25T09:19:25 (data.sh:39:main) stage -1: Data download
2022-11-25T09:19:25 (data.sh:44:main) local/data.sh: yesno directory or archive already exists in downloads. Skipping download.
2022-11-25T09:19:25 (data.sh:68:main) stage 0: Data preparation
2022-11-25T09:19:25 (data.sh:99:main) stage 1: Splitting train set into train+dev set
utils/subset_data_dir.sh: reducing #utt from 30 to 2
utils/subset_data_dir.sh: reducing #utt from 30 to 28
2022-11-25T09:19:25 (data.sh:114:main) Successfully finished. [elapsed=0s]
2022-11-25T09:19:25 (asr.sh:475:main) Skip stage 2: Speed perturbation
2022-11-25T09:19:25 (asr.sh:485:main) Stage 3: Format wav.scp: data/ -> dump/raw
utils/copy_data_dir.sh: copied data from data/train_nodev to dump/raw/org/train_nodev
utils/validate_data_dir.sh: WARNING: you have only one speaker.  This probably a bad idea.
   Search for the word 'bold' in http://kaldi-asr.org/doc/data_prep.html
   for more information.
utils/validate_data_dir.sh: Successfully validated data-directory dump/raw/org/train_nodev
2022-11-25T09:19:26 (format_wav_scp.sh:42:main) scripts/audio/format_wav_scp.sh --nj 32 --cmd run.pl --audio-format wav --fs 16k data/train_nodev/wav.scp dump/raw/org/train_nodev
2022-11-25T09:19:26 (format_wav_scp.sh:110:main) [info]: without segments
2022-11-25T09:19:28 (format_wav_scp.sh:142:main) Successfully finished. [elapsed=2s]
utils/copy_data_dir.sh: copied data from data/train_dev to dump/raw/org/train_dev
utils/validate_data_dir.sh: WARNING: you have only one speaker.  This probably a bad idea.
   Search for the word 'bold' in http://kaldi-asr.org/doc/data_prep.html
   for more information.
utils/validate_data_dir.sh: Successfully validated data-directory dump/raw/org/train_dev
2022-11-25T09:19:28 (format_wav_scp.sh:42:main) scripts/audio/format_wav_scp.sh --nj 32 --cmd run.pl --audio-format wav --fs 16k data/train_dev/wav.scp dump/raw/org/train_dev
2022-11-25T09:19:28 (format_wav_scp.sh:110:main) [info]: without segments
2022-11-25T09:19:30 (format_wav_scp.sh:142:main) Successfully finished. [elapsed=2s]
utils/copy_data_dir.sh: copied data from data/train_dev to dump/raw/org/train_dev
utils/validate_data_dir.sh: WARNING: you have only one speaker.  This probably a bad idea.
   Search for the word 'bold' in http://kaldi-asr.org/doc/data_prep.html
   for more information.
utils/validate_data_dir.sh: Successfully validated data-directory dump/raw/org/train_dev
2022-11-25T09:19:30 (format_wav_scp.sh:42:main) scripts/audio/format_wav_scp.sh --nj 32 --cmd run.pl --audio-format wav --fs 16k data/train_dev/wav.scp dump/raw/org/train_dev
2022-11-25T09:19:30 (format_wav_scp.sh:110:main) [info]: without segments
2022-11-25T09:19:32 (format_wav_scp.sh:142:main) Successfully finished. [elapsed=2s]
utils/copy_data_dir.sh: copied data from data/test_yesno to dump/raw/test_yesno
utils/validate_data_dir.sh: WARNING: you have only one speaker.  This probably a bad idea.
   Search for the word 'bold' in http://kaldi-asr.org/doc/data_prep.html
   for more information.
utils/validate_data_dir.sh: Successfully validated data-directory dump/raw/test_yesno
2022-11-25T09:19:32 (format_wav_scp.sh:42:main) scripts/audio/format_wav_scp.sh --nj 32 --cmd run.pl --audio-format wav --fs 16k data/test_yesno/wav.scp dump/raw/test_yesno
2022-11-25T09:19:32 (format_wav_scp.sh:110:main) [info]: without segments
2022-11-25T09:19:34 (format_wav_scp.sh:142:main) Successfully finished. [elapsed=2s]
2022-11-25T09:19:34 (asr.sh:588:main) Stage 4: Remove long/short data: dump/raw/org -> dump/raw
utils/copy_data_dir.sh: copied data from dump/raw/org/train_nodev to dump/raw/train_nodev
utils/validate_data_dir.sh: WARNING: you have only one speaker.  This probably a bad idea.
   Search for the word 'bold' in http://kaldi-asr.org/doc/data_prep.html
   for more information.
utils/validate_data_dir.sh: Successfully validated data-directory dump/raw/train_nodev
fix_data_dir.sh: kept all 28 utterances.
fix_data_dir.sh: old files are kept in dump/raw/train_nodev/.backup
utils/copy_data_dir.sh: copied data from dump/raw/org/train_dev to dump/raw/train_dev
utils/validate_data_dir.sh: WARNING: you have only one speaker.  This probably a bad idea.
   Search for the word 'bold' in http://kaldi-asr.org/doc/data_prep.html
   for more information.
utils/validate_data_dir.sh: Successfully validated data-directory dump/raw/train_dev
fix_data_dir.sh: kept all 2 utterances.
fix_data_dir.sh: old files are kept in dump/raw/train_dev/.backup
2022-11-25T09:19:35 (asr.sh:688:main) Stage 5: Generate character level token_list from data/train_nodev/text
[nltk_data] Downloading package averaged_perceptron_tagger to
[nltk_data]     /home/user/nltk_data...
[nltk_data]   Unzipping taggers/averaged_perceptron_tagger.zip.
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py --token_type char --input dump/raw/lm_train.txt --output data/en_token_list/char/tokens.txt --non_linguistic_symbols none --field 2- --cleaner none --g2p none --write_vocabulary true --add_symbol '<blank>:0' --add_symbol '<unk>:1' --add_symbol '<sos/eos>:-1'
2022-11-25 09:19:36,661 (tokenize_text:172) INFO: OOV rate = 0.0 %
2022-11-25T09:19:36 (asr.sh:916:main) Stage 6-8: Skip lm-related stages: use_lm=false
2022-11-25T09:19:36 (asr.sh:929:main) Stage 9: Skip ngram stages: use_ngram=false
2022-11-25T09:19:36 (asr.sh:937:main) Stage 10: ASR collect stats: train_set=dump/raw/train_nodev, valid_set=dump/raw/train_dev
2022-11-25T09:19:36 (asr.sh:987:main) Generate 'exp/asr_stats_raw_en_char/run.sh'. You can resume the process from stage 10 using this script
2022-11-25T09:19:36 (asr.sh:991:main) ASR collect-stats started... log: 'exp/asr_stats_raw_en_char/logdir/stats.*.log'
/opt/miniconda/bin/python3 /espnet/espnet2/bin/aggregate_stats_dirs.py --input_dir exp/asr_stats_raw_en_char/logdir/stats.1 --input_dir exp/asr_stats_raw_en_char/logdir/stats.2 --output_dir exp/asr_stats_raw_en_char
2022-11-25T09:19:44 (asr.sh:1038:main) Stage 11: ASR Training: train_set=dump/raw/train_nodev, valid_set=dump/raw/train_dev
2022-11-25T09:19:44 (asr.sh:1105:main) Generate 'exp/asr_train_asr_raw_en_char/run.sh'. You can resume the process from stage 11 using this script
2022-11-25T09:19:44 (asr.sh:1109:main) ASR training started... log: 'exp/asr_train_asr_raw_en_char/train.log'
2022-11-25 09:19:44,920 (launch:94) INFO: /opt/miniconda/bin/python3 /espnet/espnet2/bin/launch.py --cmd 'run.pl --name exp/asr_train_asr_raw_en_char/train.log' --log exp/asr_train_asr_raw_en_char/train.log --ngpu 1 --num_nodes 1 --init_file_prefix exp/asr_train_asr_raw_en_char/.dist_init_ --multiprocessing_distributed true -- python3 -m espnet2.bin.asr_train --use_preprocessor true --bpemodel none --token_type char --token_list data/en_token_list/char/tokens.txt --non_linguistic_symbols none --cleaner none --g2p none --valid_data_path_and_name_and_type dump/raw/train_dev/wav.scp,speech,sound --valid_data_path_and_name_and_type dump/raw/train_dev/text,text,text --valid_shape_file exp/asr_stats_raw_en_char/valid/speech_shape --valid_shape_file exp/asr_stats_raw_en_char/valid/text_shape.char --resume true --init_param --ignore_init_mismatch false --fold_length 80000 --fold_length 150 --output_dir exp/asr_train_asr_raw_en_char --config conf/train_asr.yaml --frontend_conf fs=16k --normalize=global_mvn --normalize_conf stats_file=exp/asr_stats_raw_en_char/train/feats_stats.npz --train_data_path_and_name_and_type dump/raw/train_nodev/wav.scp,speech,sound --train_data_path_and_name_and_type dump/raw/train_nodev/text,text,text --train_shape_file exp/asr_stats_raw_en_char/train/speech_shape --train_shape_file exp/asr_stats_raw_en_char/train/text_shape.char
2022-11-25 09:19:44,954 (launch:348) INFO: log file: exp/asr_train_asr_raw_en_char/train.log
2022-11-25T09:20:06 (asr.sh:1185:main) Stage 12: Decoding: training_dir=exp/asr_train_asr_raw_en_char
2022-11-25T09:20:06 (asr.sh:1213:main) Generate 'exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/run.sh'. You can resume the process from stage 12 using this script
2022-11-25T09:20:06 (asr.sh:1270:main) Decoding started... log: 'exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/train_dev/logdir/asr_inference.*.log'
2022-11-25T09:20:14 (asr.sh:1286:main) Calculating RTF & latency... log: 'exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/train_dev/logdir/calculate_rtf.log'
2022-11-25T09:20:15 (asr.sh:1270:main) Decoding started... log: 'exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/test_yesno/logdir/asr_inference.*.log'
2022-11-25T09:20:24 (asr.sh:1286:main) Calculating RTF & latency... log: 'exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/test_yesno/logdir/calculate_rtf.log'
2022-11-25T09:20:25 (asr.sh:1312:main) Stage 13: Scoring
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py -f 2- --input - --output - --token_type char --non_linguistic_symbols none --remove_non_linguistic_symbols true --cleaner none
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py -f 2- --input - --output - --token_type char --non_linguistic_symbols none --remove_non_linguistic_symbols true
2022-11-25T09:20:26 (asr.sh:1413:main) Write cer result in exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/train_dev/score_cer/result.txt
|  SPKR     |  # Snt   # Wrd   |  Corr       Sub      Del       Ins       Err    S.Err   |
|  Sum/Avg  |     2       52   |  90.4       7.7      1.9      53.8      63.5    100.0   |
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py -f 2- --input - --output - --token_type word --non_linguistic_symbols none --remove_non_linguistic_symbols true --cleaner none
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py -f 2- --input - --output - --token_type word --non_linguistic_symbols none --remove_non_linguistic_symbols true
2022-11-25T09:20:28 (asr.sh:1413:main) Write wer result in exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/train_dev/score_wer/result.txt
|  SPKR     |  # Snt   # Wrd   |  Corr       Sub      Del       Ins       Err    S.Err   |
|  Sum/Avg  |     2       16   |  87.5      12.5      0.0      56.3      68.8    100.0   |
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py -f 2- --input - --output - --token_type char --non_linguistic_symbols none --remove_non_linguistic_symbols true --cleaner none
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py -f 2- --input - --output - --token_type char --non_linguistic_symbols none --remove_non_linguistic_symbols true
2022-11-25T09:20:30 (asr.sh:1413:main) Write cer result in exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/test_yesno/score_cer/result.txt
|  SPKR     |  # Snt    # Wrd  |  Corr       Sub       Del       Ins      Err     S.Err   |
|  Sum/Avg  |    30       835  |  89.2       6.9       3.8      24.1     34.9     100.0   |
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py -f 2- --input - --output - --token_type word --non_linguistic_symbols none --remove_non_linguistic_symbols true --cleaner none
/opt/miniconda/bin/python3 /espnet/espnet2/bin/tokenize_text.py -f 2- --input - --output - --token_type word --non_linguistic_symbols none --remove_non_linguistic_symbols true
2022-11-25T09:20:31 (asr.sh:1413:main) Write wer result in exp/asr_train_asr_raw_en_char/decode_asr_model_valid.acc.ave/test_yesno/score_wer/result.txt
|  SPKR     |  # Snt    # Wrd  |  Corr       Sub       Del       Ins      Err     S.Err   |
|  Sum/Avg  |    30       240  |  87.5      10.4       2.1      27.1     39.6     100.0   |
fatal: not a git repository (or any parent up to mount point /espnet)
Stopping at filesystem boundary (GIT_DISCOVERY_ACROSS_FILESYSTEM not set).
fatal: not a git repository (or any parent up to mount point /espnet)
Stopping at filesystem boundary (GIT_DISCOVERY_ACROSS_FILESYSTEM not set).
ls: cannot access 'exp/asr_train_asr_raw_en_char/*/*/score_wer/scoring/*.filt.sys': No such file or directory
<!-- Generated by scripts/utils/show_asr_result.sh -->
# RESULTS
## Environments
- date: `Fri Nov 25 09:20:31 UTC 2022`
- python version: `3.7.4 (default, Aug 13 2019, 20:35:49)  [GCC 7.3.0]`
- espnet version: `espnet 202209`
- pytorch version: `pytorch 1.10.1+cu111`
- Git hash: ``
  - Commit date: ``

## asr_train_asr_raw_en_char
### WER

|dataset|Snt|Wrd|Corr|Sub|Del|Ins|Err|S.Err|
|---|---|---|---|---|---|---|---|---|
|decode_asr_model_valid.acc.ave/test_yesno|30|240|87.5|10.4|2.1|27.1|39.6|100.0|
|decode_asr_model_valid.acc.ave/train_dev|2|16|87.5|12.5|0.0|56.3|68.8|100.0|

### CER

|dataset|Snt|Wrd|Corr|Sub|Del|Ins|Err|S.Err|
|---|---|---|---|---|---|---|---|---|
|decode_asr_model_valid.acc.ave/test_yesno|30|835|89.2|6.9|3.8|24.1|34.9|100.0|
|decode_asr_model_valid.acc.ave/train_dev|2|52|90.4|7.7|1.9|53.8|63.5|100.0|

### TER

|dataset|Snt|Wrd|Corr|Sub|Del|Ins|Err|S.Err|
|---|---|---|---|---|---|---|---|---|
2022-11-25T09:20:33 (asr.sh:1434:main) Stage 14: Pack model: exp/asr_train_asr_raw_en_char/asr_train_asr_raw_en_char_valid.acc.ave.zip
/opt/miniconda/lib/python3.7/zipfile.py:1473: UserWarning: Duplicate name: 'exp/asr_train_asr_raw_en_char/RESULTS.md'
  return self._open_to_write(zinfo, force_zip64=force_zip64)
adding: meta.yaml
adding: exp/asr_train_asr_raw_en_char/config.yaml
adding: exp/asr_train_asr_raw_en_char/7epoch.pth
adding: exp/asr_stats_raw_en_char/train/feats_stats.npz
adding: exp/asr_train_asr_raw_en_char/RESULTS.md
adding: exp/asr_train_asr_raw_en_char/RESULTS.md
adding: exp/asr_train_asr_raw_en_char/images
adding: exp/asr_train_asr_raw_en_char/images/forward_time.png
adding: exp/asr_train_asr_raw_en_char/images/loss.png
adding: exp/asr_train_asr_raw_en_char/images/acc.png
adding: exp/asr_train_asr_raw_en_char/images/loss_ctc.png
adding: exp/asr_train_asr_raw_en_char/images/backward_time.png
adding: exp/asr_train_asr_raw_en_char/images/wer.png
adding: exp/asr_train_asr_raw_en_char/images/cer.png
adding: exp/asr_train_asr_raw_en_char/images/gpu_max_cached_mem_GB.png
adding: exp/asr_train_asr_raw_en_char/images/optim_step_time.png
adding: exp/asr_train_asr_raw_en_char/images/cer_ctc.png
adding: exp/asr_train_asr_raw_en_char/images/iter_time.png
adding: exp/asr_train_asr_raw_en_char/images/optim0_lr0.png
adding: exp/asr_train_asr_raw_en_char/images/train_time.png
adding: exp/asr_train_asr_raw_en_char/images/loss_att.png
Generate: exp/asr_train_asr_raw_en_char/asr_train_asr_raw_en_char_valid.acc.ave.zip
2022-11-25T09:20:35 (asr.sh:1522:main) Skip the uploading stage
2022-11-25T09:20:35 (asr.sh:1574:main) Skip the uploading to HuggingFace stage
2022-11-25T09:20:35 (asr.sh:1577:main) Successfully finished. [elapsed=70s]
run.sh done.
```
정상적으로 데이터 준비, 전처리, 모델 훈련을 마치고 CER / WER로 evaluation까지 수행한 뒤 스크립트가 종료되는 것을 확인하였다.  

### 첨언  
ESPnet의 레시피 내 리소스 구성 및 훈련 스크립트를 이해하기 위해서는, KALDI에 대한 사전 지식이 어느 정도 필요하다. 
다음 포스팅 시 KALDI에 대한 내용을 조금 정리해볼까 한다.