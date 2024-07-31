# Exadata Flashcache Flush Procedure

모든 작업은 DB#01에서 진행 될 수 있도록 하였습니다.
## 1. 작업 전 상태확인
1) Flashcache 상태 확인
   Status **normal** 상태 확인
```shell
# dcli -g /root/cell_group -l root "cellcli -e list flashcache attributes name,size,status"
pcposcl01: pcposcl01_FLASHCACHE	 11.643218994140625T	 normal
pcposcl02: pcposcl02_FLASHCACHE	 11.643218994140625T	 normal
pcposcl03: pcposcl03_FLASHCACHE	 11.643218994140625T	 normal
```
## 2. Exadata Flashcache Flush
1) Flashcache Flush (Rolling) : DB#01에서 진행
   각 CELL Node별 Rolling Flashcache Flush 진행 완료 후 다음 명령 수행
   (CELL hostname : pcposcl01,pcposcl02,pcposcl03)
```shell
# nohup dcli -c pcposcl01 -l root "cellcli -e alter flashcache all flush" &
# nohup dcli -c pcposcl02 -l root "cellcli -e alter flashcache all flush" &
# nohup dcli -c pcposcl03 -l root "cellcli -e alter flashcache all flush" &
```
2)  Flashcache Flush (Non-Rolling) : DB#01에서 진행
```shell
# nohup dcli -c /root/cell_group -l root "cellcli -e alter flashcache all flush" &
```
3) Flashcache Flushing 상태 조회
   Status **normal - flushing** 상태 확인,  완료 시 **normal - flushed** 로 표시
   FC USED, DIRTY, ALLOCATED Size 확인
```shell
# dcli -g /root/cell_group -l root "cellcli -e list flashcache attributes name, size, status"

pcposcl01: pcposcl01_FLASHCACHE	 11.643218994140625T	 normal - flushing
pcposcl02: pcposcl02_FLASHCACHE	 11.643218994140625T	 normal - flushing
pcposcl03: pcposcl03_FLASHCACHE	 11.643218994140625T	 normal - flushing

# dcli -g /root/cell_group -l root cellcli -e list metriccurrent FC_BY_USED,FC_BY_DIRTY,FC_BY_ALLOCATED

pcposcl01: FC_BY_USED     	 FLASHCACHE	 1,790,941 MB
pcposcl01: FC_BY_DIRTY    	 FLASHCACHE	 55,328 MB
pcposcl01: FC_BY_ALLOCATED	 FLASHCACHE	 1,841,038 MB
pcposcl02: FC_BY_USED     	 FLASHCACHE	 1,795,176 MB
pcposcl02: FC_BY_DIRTY    	 FLASHCACHE	 56,929 MB
pcposcl02: FC_BY_ALLOCATED	 FLASHCACHE	 1,845,244 MB
pcposcl03: FC_BY_USED     	 FLASHCACHE	 1,789,580 MB
pcposcl03: FC_BY_DIRTY    	 FLASHCACHE	 57,016 MB
pcposcl03: FC_BY_ALLOCATED	 FLASHCACHE	 1,839,719 MB
```
   30초 단위 모니터링을 위해 아래 Command 이용
```shell
# watch -n 30 "dcli -g /root/cell_group -l root "cellcli -e list flashcache attributes name, size, status"

# watch -n 30 "dcli -g /root/cell_group -l root cellcli -e list metriccurrent FC_BY_USED,FC_BY_DIRTY,FC_BY_ALLOCATED"
```
4) Flashcache Flushing 완료 진행
   Flushing 완료 후 Status **normal** 상태 확인
```shell
# dcli -g /root/cell_group -l root "cellcli -e list flashcache attributes name, size, status"
pcposcl01: pcposcl01_FLASHCACHE	 11.643218994140625T	 normal - flushed
pcposcl02: pcposcl02_FLASHCACHE	 11.643218994140625T	 normal - flushed
pcposcl03: pcposcl03_FLASHCACHE	 11.643218994140625T	 normal - flushed

# dcli -g /root/cell_group -l root "cellcli -e alter flashcache all cancel flush"
pcposcl01: Flash cache pcposcl01_FLASHCACHE altered successfully
pcposcl02: Flash cache pcposcl02_FLASHCACHE altered successfully
pcposcl03: Flash cache pcposcl03_FLASHCACHE altered successfully

# dcli -g /root/cell_group -l root "cellcli -e list flashcache attributes name, size, status"
pcposcl01: pcposcl01_FLASHCACHE	 11.643218994140625T	 normal
pcposcl02: pcposcl02_FLASHCACHE	 11.643218994140625T	 normal
pcposcl03: pcposcl03_FLASHCACHE	 11.643218994140625T	 normal
```
