# grib2-to-bin-



```shell
 #!/bin/tcsh
  2
###set the original time###
  3 set yy=2012
  4 set mm=06
  5 set dd=09
  6 set hh=12
  7 set nn=00
  8
  9 echo ' ncep grib2 data to cress ' $yy $mm $dd $hh $nn
###declare varibal###
 10 set wgrib2=/usr/bin/wgrib2
 11 set fortran=/work1/summer/junyou/20120610-20160612-6HR-DAT
 12 set gribdata=/work1/summer/junyou/20120610-20160612-6HR-DAT
 13 set gpvout=/work1/summer/junyou/20120610-20160612-6HR-DAT/gpv_file2
 14
 15 mkdir -p $gpvout
 16 rm -f $gpvout/*temp*
 17 cd $gribdata
 18
###set the time ###
 19 if($mm == 01)set idmax=31
 20 if($mm == 02)set idmax=28
 21 if($mm == 03)set idmax=31
 22 if($mm == 04)set idmax=30
 23 if($mm == 05)set idmax=31
 24 if($mm == 06)set idmax=30
 25 if($mm == 07)set idmax=31
 26 if($mm == 08)set idmax=31
 27 if($mm == 09)set idmax=30
 28 if($mm == 10)set idmax=31
 29 if($mm == 11)set idmax=30
 30 if($mm == 12)set idmax=31
 31  if($mm == 02 && $yy == 1988)set idmax=29
 32  if($mm == 02 && $yy == 1992)set idmax=29
 33  if($mm == 02 && $yy == 1996)set idmax=29
 34  if($mm == 02 && $yy == 2000)set idmax=29
 35  if($mm == 02 && $yy == 2004)set idmax=29
 36  if($mm == 02 && $yy == 2008)set idmax=29
 37  if($mm == 02 && $yy == 2012)set idmax=29
 38  if($mm == 02 && $yy == 2016)set idmax=29
 39  if($mm == 02 && $yy == 2020)set idmax=29
 40
 41 ########## count time ##########
 42
 43 foreach h3 (00 06 12 18 24 30 36 42 48 54 60 66 72 78 84 90 96)###12 18 24 30 36 42 48 54 60 66 72) ### 78 84 90 96 102 108 114 120 126 132 138 144 150 1    56 162 168 174 180 186 192)
 44
 45  set cy=$yy
 46  set cm=$mm
 47  set cd=$dd
 48  set h4=$h3
 49
 50  if( $cm == 01 )set cm = 1 ; if( $cm == 02 )set cm = 2
 51  if( $cm == 03 )set cm = 3 ; if( $cm == 04 )set cm = 4
 52  if( $cm == 05 )set cm = 5 ; if( $cm == 06 )set cm = 6
 53  if( $cm == 07 )set cm = 7 ; if( $cm == 08 )set cm = 8 ; if( $cm == 09 )set cm = 9
 54
 55  if( $cd == 01 )set cd = 1 ; if( $cd == 02 )set cd = 2
 56  if( $cd == 03 )set cd = 3 ; if( $cd == 04 )set cd = 4
 57  if( $cd == 05 )set cd = 5 ; if( $cd == 06 )set cd = 6
 58  if( $cd == 07 )set cd = 7 ; if( $cd == 08 )set cd = 8 ; if( $cd == 09 )set cd = 9
 59
 60  if( $h4 == 00 )set h4 = 0 ; if( $h4 == 03 )set h4 = 3
 61  if( $h4 == 06 )set h4 = 6 ; if( $h4 == 09 )set h4 = 9
 62
 63   @ chh = $h4 + $hh
 64   while($chh >= 24)
 65    @ chh = $chh - 24
 66    @ cd = $cd + 1
 67    while($cd > $idmax)
 68     @ cd = $cd - $idmax
 69     @ cm = $cm + 1
 70     while($cm > 12)
 71      @ cm = $cm - 12
 72      @ cy = $cy + 1
 73     end
 74    end
 75   end
 76
 77 # echo 'counting time:' $cm $cd $chh
 78  @ cm = $cm + 0
 79  @ cd = $cd + 0
 80  @ chh = $chh + 0
 81
 82  if($cm < 10)set cm='0'$cm
 83  if($cd < 10)set cd='0'$cd
 84  if($chh < 10)set chh='0'$chh
 85
 86 ### eg. fnl_20120610_00_00.grib2
 87 ### eg. fnl_20120610_06_00.grib2
 88  set fin='fnl_'$cy$cm$cd'_'$chh'_00.grib2'
 89  set sstbin='data.sst'$cy$cm$cd$chh$nn'.bin'
 90  set sstgrb='temp.sst'$cy$cm$cd$chh$nn'.grb2'
 91  set foutbin='data.gpv'$cy$cm$cd$chh$nn'.bin'
 92  set tempbin=$cy$cm$cd$chh$nn'.bin'
 93  set foutgrd='data.gpv'$cy$cm$cd$chh$nn'.grd'
 94  echo ' '
 95  echo 'process :' $fin '=>'  $foutbin
 96  echo 'data time:' $cy $cm'/'$cd $chh':'$nn
 97
 98 ########## get sst data from grib2 ##########
 99
100 set keyword=':TMP:surface:'
101 echo 'sst( '$keyword' ):' $fin '=>' $sstbin
102
103  $wgrib2 $fin | egrep "$keyword" | $wgrib2 -i -g2clib 0  $fin -small_grib 95:145 0:50 $gpvout/$sstgrb > $gpvout/temp.sst.log
104  $wgrib2 $gpvout/$sstgrb | $wgrib2 -i -no_header -g2clib 0 $gpvout/$sstgrb -bin $gpvout/$sstbin > $gpvout/temp.sst.log
105
106  rm -f $gpvout/$sstgrb
107
108 #################################################
109
110 ########## get grid data form grib2 ##########
111
112 ### total 27 levels including 1 surface level(slev) and 26 air z levels(ulvl)
113 set slev=('surface' '10 m above ground' '10 m above ground' 'surface' '2 m above ground' '2 m above ground')
114 set ulvl=(1000 975 950 925 900 850 800 750 700 650 600 550 500 450 400 350 300 250 200 150 100 70 50 30 20) ### 10)
115
116 ### 6 gpv data variables
117 set var=(HGT UGRD VGRD PRES TMP RH)
118
119 set fout1=temp1.grb2
120 set fout2=temp2.grb2
121 touch $gpvout/$fout1
122
123 set ivar = 1
124 while($ivar <= $#var)
125 # set keyword = `echo ':'$var[$ivar]':'$slev[$ivar]':'`   # get suface variable
126 # $wgrib2 $fin | egrep "$keyword" | $wgrib2 -i -g2clib 0 $fin -small_grib 95:145 0:50 $gpvout/$fout2 > $gpvout/temp.log
127 # cat $gpvout/$fout2 >> $gpvout/$fout1
128
129  set iulvl = 1
130  while($iulvl <= $#ulvl)
131   set keyword = ':'$var[$ivar]':'$ulvl[$iulvl]' mb:'   #get air z levels variable
132   $wgrib2  $fin | egrep "$keyword" | $wgrib2 -i -g2clib 0 $fin -small_grib 95:145 0:50 $gpvout/$fout2 > $gpvout/temp.log
133   cat $gpvout/$fout2 >> $gpvout/$fout1
134   @ iulvl = $iulvl + 1
135  end
136  @ ivar = $ivar + 1
137 end
138
139 rm -f $gpvout/$fout2
140
141 #################################################
142
143 ########## covert grib2 to binary ##########
144 $wgrib2 $gpvout/$fout1 | $wgrib2 -i -no_header -g2clib 0 $gpvout/$fout1 -bin $gpvout/$tempbin > $gpvout/temp.grid.log
145 mv $gpvout/$fout1 $gpvout/$foutgrd
146
147 ########## fill air pressure data by fortran ##########
148 cd $gpvout
149 $fortran/a.out << EOF
150 $tempbin
151 $foutbin
152 EOF
153 #$tempbin
154 #$foutbin
155 #################################################
156
157 ########## remove temp data ##########
158 rm -f $foutgrd $tempbin temp.log
159 cd $gribdata
160
161 #################################################
162
163 end   # foreach h3
164
165 ########## gpv / sst ctl ##########
166 ### HGT UGRD VGRD PRES TMP RH ### SST ###
167 cd $gpvout
168 if( $mm == 01)set am = Jan ; if( $mm == 02)set am = Feb
169 if( $mm == 03)set am = Mar ; if( $mm == 04)set am = Apr
170 if( $mm == 05)set am = May ; if( $mm == 06)set am = Jun
171 if( $mm == 07)set am = Jul ; if( $mm == 08)set am = Aug
172 if( $mm == 09)set am = Sep ; if( $mm == 10)set am = Oct
173 if( $mm == 11)set am = Nov ; if( $mm == 12)set am = Dec
174 cat > data.gpv.ctl << EOF_GPV
175 dset  ^data.gpv%y4%m2%d2%h2%n2.bin
176 title  gpv 1.0 data
177 undef  -1.e35
178 options  template
179 xdef  51  linear   95.0  1.00
180 ydef  51  linear    0.0  1.00
181 zdef  25  levels  $ulvl
182 tdef   17  linear  ${hh}z$dd$am$yy  6hr
183 vars  6
184 hgt  25 0  geopotential height (gpm;m)
185 ugrd 25 0  u-wind (m/s)
186 vgrd 25 0  v-wind (m/s)
187 pres 25 0  pressure (Pa)
188 tmp  25 0  temperature (Kelvin)
189 rh   25 0  relative humidity (%)
190 endvars
191 EOF_GPV
192
193 cat > data.sst.ctl << EOF_SST
194 dset ^data.sst%y4%m2%d2%h2%n2.bin
195 title ncep sst 1.0
196 undef -1.e35
197 options  template
198 xdef  51 linear  95.0  1.00
199 ydef  51 linear   0.0  1.00
200 zdef   1 levels  2001
201 tdef   17  linear ${hh}z$dd$am$yy  6hr
202 vars   1
203 sst  0 0 sea surface temperature (deg K)
204 endvars
205 EOF_SST
```
