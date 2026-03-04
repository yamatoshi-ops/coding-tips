---
1. **dq平均（DCの (I_d, I_q)）が同じでも、dqに3次リップルが乗ると平均トルクは下がり得るか？**
2. **「奇数次でdqが変動」すると、平均ベクトルが同じでも平均トルクは変わるのか？**

結論を先に言います。

* **はい、下がり得ます。**
* ただし「奇数次だから」というより、**dqリップルが作る“2次効果（平均化しても残る項）”**と、**電圧制限/損失/磁束の非線形**が絡むと平均トルクが変わります。3%は十分起こり得るレンジです。

以下、メカニズムを分解します（実務で当たりを付けられる形で）。

---

## 1) dqリップルがあっても平均トルクが変わらない“理想条件”

理想IPMSM（パラメータ一定、磁気線形、電圧は理想印加、角度も理想、損失なし）で

[
T(t)=\frac{3}{2}p\Big(\psi_f, i_q(t)+ (L_d-L_q), i_d(t)i_q(t)\Big)
]

と書けます。

ここで
[
i_d(t)=I_d+\tilde i_d(t),\quad i_q(t)=I_q+\tilde i_q(t)
]
（(\tilde{}) はゼロ平均のリップル）

平均を取ると

[
\overline{T}= \frac{3}{2}p\Big(\psi_f I_q + (L_d-L_q)\big(I_d I_q + \overline{\tilde i_d \tilde i_q}\big)\Big)
]

ポイントはここだけです：

* **平均トルクが変わるのは** (\overline{\tilde i_d \tilde i_q}) **が非ゼロのとき**
  （＝ dとqのリップルが相関しているとき）
* もし (\tilde i_d,\tilde i_q) が互いに独立で、相関ゼロなら、**平均はほぼ (I_d,I_q) で決まる**

つまり「奇数次かどうか」は本質ではなく、**dqリップルの相関があるか**が1つ目の本質です。

---

## 2) それでも平均トルクが落ちる“現実的”な主要因（3%級が出るところ）

あなたの現象（抑制OFFで3次・9次が出て、dqにも3次が乗り、軸トルクが約-3%）は、次のどれか（複合が多い）で説明できます。

### (A) **dqリップル → 銅損増 → 速度/トルクの実効が落ちる**

電流RMSが増えます。

[
I_{\mathrm{rms}}^2 = I_d^2+I_q^2+\overline{\tilde i_d^2}+\overline{\tilde i_q^2}
]

抑制OFFでは (\overline{\tilde i^2}) が増えるので **銅損 (P_{cu}=3R_s I_{\mathrm{rms}}^2)** が増えます。

* トルク指令一定の閉ループでも、電圧余裕や温度上昇→抵抗上昇→電流制御誤差が増える
* トルクが「推定トルク」ではなく「軸トルク（動力計）」だと、損失増がそのまま効きます

→ **平均 (I_d,I_q) が同じでも、軸で見たトルクが目減り**は普通に起こります。

※ これは“電磁トルクが同じでも”軸トルクが落ちる代表例です。

---

### (B) **電流リップル × 非線形（磁気飽和・クロスサチュレーション）で平均トルクが変わる**

実機IPMは (L_d,L_q,\psi_f) が電流依存です：

[
L_d=L_d(i_d,i_q),\quad L_q=L_q(i_d,i_q),\quad \psi_f=\psi_f(i_d,i_q)
]

このときトルクは線形ではなく、リップルを入れると

* **平均値のまわりの二次項**（曲率）が残って、(\overline{T}) が変わります。

直感的には
「同じ平均でも、凸/凹な関数に揺れを入れると平均がずれる（Jensenの不等式）」です。

IPMのMTPA近傍はこの“曲率効果”が出やすく、**数%級のズレは十分あり得ます**。

---

### (C) **電圧制限（飽和）により、i_d/i_q が瞬時的に追従できず平均トルクが落ちる**

3次でdqが揺れるということは、電流制御器がその周波数の電圧を出そうとします。

しかし実際は

* DCリンク
* 変調限界
* デッドタイム/電圧誤差
* 両インバータの共通モード戦略

で **瞬時電圧が足りない区間**が出ます。

そのとき

* リップル成分だけでなく、**基本波の有効q電圧も食われる**
* 結果として **(i_q) の実効が少し下がる／位相がずれる**
* “平均のdq電流ベクトル”をロガーが同じと見ていても、実は同期抽出の仕方で見えていない差が残る

→ 軸トルク低下に直結します。

---

### (D) **トルク推定／角度／観測のバイアス**

抑制OFFでゼロ相・共通モードが増えると、

* 電流検出のオフセットが動く
* ADCの同相ノイズが増える
* 推定角のPLL/EKFにリップルが乗る

などで、結果として **dqが“見かけ上”揺れたり、制御が余計な仕事をして損する**ことが起きます。

---

## 3) 質問への直接回答

> 平均の電流ベクトルはONOFFで全く変わらなくても、dq軸電流が3次で変動しているOFFのケースは、平均トルクとしては、目減りするか？

**目減りします（し得ます）。**
主因は **(A)銅損増** と **(B)磁気非線形による二次効果**、次点で **(C)電圧飽和** です。

> dq軸電流が奇数次で変動すると、平均の電流ベクトルが同じでも平均トルクはかわってしまう？

「奇数次だから」ではなく、

* (\overline{\tilde i_d \tilde i_q}\neq 0)（相関がある）
* あるいはトルクが非線形（(L_d,L_q,\psi_f) が電流依存）
* あるいは損失・電圧制限が効く

のいずれかで **変わります**。あなたの状況はこの条件を満たす可能性が高いです。

---

## 4) いまの情報で最短に“犯人”を絞るチェック（おすすめ順）

追加の質問ではなく、やると切れる観測です。

1. **抑制ON/OFFで RMS電流（相電流でもdqでも）を比較**

   * (I_{\rm rms}) が増えていれば銅損起因が濃厚（まずここ）
2. **(\overline{\tilde i_d \tilde i_q}) を計算**（ログから可能）

   * 非ゼロなら、リップル相関で平均トルクがズレる土台あり
3. **電圧指令の飽和率（SVPWMのmodulation index、電圧クリップ回数）**

   * OFFで飽和が増えるなら(C)が濃厚
4. **Ld/Lq/ψの電流依存（マップ or 推定）**

   * MTPA点での曲率が強いなら(B)が濃厚

---

---

# 1. Clarke変換の定義

相電流

[
\mathbf{i}_{abc}=
\begin{bmatrix}
i_a\
i_b\
i_c
\end{bmatrix}
]

Clarke変換

[
\begin{bmatrix}
i_\alpha\
i_\beta\
i_0
\end{bmatrix}
=============

\begin{bmatrix}
\frac{2}{3} &-\frac{1}{3} &-\frac{1}{3}\
0&\frac{1}{\sqrt3}&-\frac{1}{\sqrt3}\
\frac13&\frac13&\frac13
\end{bmatrix}
\begin{bmatrix}
i_a\
i_b\
i_c
\end{bmatrix}
]

つまり

[
i_\alpha=\frac{2}{3}i_a-\frac13 i_b-\frac13 i_c
]

[
i_\beta=\frac{1}{\sqrt3}(i_b-i_c)
]

[
i_0=\frac13(i_a+i_b+i_c)
]

---

# 2. 純零相電流のケース

純零相とは

[
i_a=i_b=i_c=i_z
]

これを代入します。

### α

[
i_\alpha=\frac23 i_z-\frac13 i_z-\frac13 i_z
]

[
i_\alpha=0
]

### β

[
i_\beta=\frac{1}{\sqrt3}(i_z-i_z)
]

[
i_\beta=0
]

### 0軸

[
i_0=\frac13(3i_z)=i_z
]

つまり

[
(i_\alpha,i_\beta)=(0,0)
]

**純零相は完全に0軸へ分離されます。**

---

# 3. ゼロ相が「崩れた」ケース

あなたの状況（デュアルインバータ電圧ばらつき）では

[
i_a=i_z
]

[
i_b=i_z+\epsilon_1
]

[
i_c=i_z+\epsilon_2
]

と書けます。

---

## α成分

[
i_\alpha
=\frac23 i_z-\frac13(i_z+\epsilon_1)-\frac13(i_z+\epsilon_2)
]

展開

[
i_\alpha=
\frac23 i_z-\frac13 i_z-\frac13 i_z
-\frac13\epsilon_1-\frac13\epsilon_2
]

[
i_\alpha=-\frac13(\epsilon_1+\epsilon_2)
]

---

## β成分

[
i_\beta
=\frac{1}{\sqrt3}[(i_z+\epsilon_1)-(i_z+\epsilon_2)]
]

[
i_\beta=\frac{1}{\sqrt3}(\epsilon_1-\epsilon_2)
]

---

# 4. 結果

[
i_\alpha=-\frac13(\epsilon_1+\epsilon_2)
]

[
i_\beta=\frac{1}{\sqrt3}(\epsilon_1-\epsilon_2)
]

重要なのは

[
i_z
]

は **完全に消えている**ことです。

つまり

**αβに現れるのは純零相ではなく**

[
(\epsilon_1,\epsilon_2)
]

すなわち

**相電流の不一致成分**

です。

---

# 5. 物理的意味

理想

```
ia = ib = ic
```

→ 完全零相
→ αβ = 0

実機

```
ia ≠ ib ≠ ic
```

→ 零相 + 正相 + 逆相

つまり

[
i_{abc}=i^{(0)}+i^{(+)}+i^{(-)}
]

Clarkeは

* 正相
* 逆相

だけをαβへ写します。

---

# 6. 高調波で見るとどうなるか

例えば3次電流

理想

[
i_a=A\sin(3\omega t)
]

[
i_b=A\sin(3\omega t)
]

[
i_c=A\sin(3\omega t)
]

→ 純零相
→ αβ = 0

---

しかし

電圧ばらつきで

[
i_b=A\sin(3\omega t+\delta_1)
]

[
i_c=A\sin(3\omega t+\delta_2)
]

となると

[
\epsilon_1,\epsilon_2
]

が

[
3\omega
]

で振動するため

[
i_\alpha,i_\beta
]

にも

[
3\omega
]

が現れます。

---

# 7. まとめ

あなたの理解を数式で書くと

### 純零相

[
i_a=i_b=i_c
]

[
i_\alpha=i_\beta=0
]

---

### 崩れた零相

[
i_a=i_z
]

[
i_b=i_z+\epsilon_1
]

[
i_c=i_z+\epsilon_2
]

[
i_\alpha=-\frac13(\epsilon_1+\epsilon_2)
]

[
i_\beta=\frac{1}{\sqrt3}(\epsilon_1-\epsilon_2)
]

---

つまり

**αβに出るのは**

**零相そのものではなく**

**零相の崩れ（不平衡成分）**

です。

---

要点はこれです：

* **3次（3ω）の“崩れた零相”が Clarke で αβ に現れる**
* Park（回転座標変換）は **周波数を −ω シフト**するので
  **αβの3ω成分は dq で 2ω 成分になる**
* トルクは (i_d,i_q) の一次項＋積の項で決まるため、**dqに2ωの電流リップルが乗るとトルクにも2ωリップルが出る**

以下、紙で追える式で一本にします。

---

# 1) dq電流に 2ω リップルが乗る形（ここが出発点）

3次由来の成分が dq で 2ω になったとします（前段の議論の結果）。

[
\begin{aligned}
i_d(t) &= I_d + \hat i_d \cos(2\omega t+\alpha)\
i_q(t) &= I_q + \hat i_q \sin(2\omega t+\beta)
\end{aligned}
]

（位相は一般にずれます。sin/cosの形は任意なので、cos/cosでもOKです。）

---

# 2) IPMSMの電磁トルク式

（損失や高周波磁束は無視した、標準のdqモデル）

[
T_e(t)=\frac{3}{2}p\Big(\psi_f, i_q(t) + (L_d-L_q), i_d(t)i_q(t)\Big)
]

---

# 3) 代入して周波数成分を分解

## 3.1 一次項（PMトルク）

[
T_{\mathrm{pm}}(t)=\frac{3}{2}p,\psi_f, i_q(t)
]

[
=\frac{3}{2}p,\psi_f\left(I_q+\hat i_q \sin(2\omega t+\beta)\right)
]

よって **2ωのトルクリップル**がそのまま出ます：

[
\boxed{T_{\mathrm{pm}}(t)=T_{\mathrm{pm,dc}}+\underbrace{\frac{3}{2}p,\psi_f,\hat i_q \sin(2\omega t+\beta)}_{2\omega\ \text{リップル}}}
]

---

## 3.2 積の項（リラクタンストルク）

[
T_{\mathrm{rel}}(t)=\frac{3}{2}p,(L_d-L_q), i_d(t)i_q(t)
]

積を展開：

[
i_d i_q
=======

\left(I_d+\hat i_d \cos(2\omega t+\alpha)\right)
\left(I_q+\hat i_q \sin(2\omega t+\beta)\right)
]

# [

I_d I_q
+I_d\hat i_q \sin(2\omega t+\beta)
+I_q\hat i_d \cos(2\omega t+\alpha)
+\hat i_d\hat i_q \cos(2\omega t+\alpha)\sin(2\omega t+\beta)
]

ここで最後の項は恒等式で

[
\cos A,\sin B=\frac{1}{2}\big(\sin(B+A)+\sin(B-A)\big)
]

なので、

* (\sin((2\omega t)+(2\omega t))=\sin(4\omega t)) → **4ω**
* (\sin((2\omega t)-(2\omega t))=\sin(\text{定数})) → **DC（平均ずれ）**

になります。

したがって

[
\boxed{i_d i_q \text{ の中には } 2\omega\ \text{項が必ず出る}}
]

具体的に 2ω 部分だけ抜くと：

[
(i_d i_q)_{2\omega}
===================

I_d\hat i_q \sin(2\omega t+\beta)
+I_q\hat i_d \cos(2\omega t+\alpha)
]

よって

[
\boxed{T_{\mathrm{rel}}(t)=T_{\mathrm{rel,dc}}+
\frac{3}{2}p,(L_d-L_q)
\left[
I_d\hat i_q \sin(2\omega t+\beta)
+I_q\hat i_d \cos(2\omega t+\alpha)
\right]
+\text{(DC, 4}\omega\text{)}}
]

---

# 4) まとめ：なぜ「2ωトルクリップル」になるのか

上の (3.1)(3.2) から、

[
\boxed{\text{dqに2}\omega\text{の電流リップルがあると、トルクは必ず2}\omega\text{で揺れる}}
]

理由は単純で：

* トルク式が (i_q) を**線形**に含む（PMトルク）
* さらに (i_d i_q) を含むので、どちらかに2ωがあれば **積の展開で2ωが出る**

---

# 5) ここをあなたの現象（3%平均トルク差）に接続する重要点

上の式で見える通り：

* 2ωリップル自体は「平均0」なので、理想線形なら**平均トルクは変わらない**こともあります
* しかし現実は

  * (L_d,L_q,\psi_f) が電流で変わる（飽和・クロスサチュ）
  * 電圧飽和で2ωの補償ができず、基本波の有効 (I_q) が少し食われる
  * RMS増で銅損増→軸トルク減
  * (\hat i_d\hat i_q) 項が **DCずれ**（(\sin(\beta-\alpha))）を生み得る

ので、**平均トルクが下がる**（あなたの3%）につながります。

---

## 最短の確認計算（ログがあればすぐできる）

* (i_d,i_q) から 2ω成分の振幅 (\hat i_d,\hat i_q) と位相差 (\beta-\alpha) を抽出
* (\overline{\tilde i_d \tilde i_q})（相関）を直接計算
  (\overline{\tilde i_d \tilde i_q}\neq 0) なら、リラクタンストルク平均がずれる土台が確定

---

