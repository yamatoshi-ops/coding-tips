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

結論から言うと、**「3次だから平均が≠0、6次だから平均0」というわけではありません。**
平均トルクが変わるかどうかは **“次数”ではなく、( \tilde i_d ) と ( \tilde i_q ) の位相関係（相関）**で決まります。

ただし、あなたのケース（**3次零相 → dqで2ω**）では、**dqベクトルが小さく回る運動**になるため、**平均がズレやすい**という特徴は確かにあります。以下で整理します。

---

# 1. dqベクトルが小さく回るとは何か（数学）

電流ベクトルを

[
\mathbf{i}(t) =
\begin{bmatrix}
i_d(t)\
i_q(t)
\end{bmatrix}
]

として

[
i_d = I_d + r\cos(\Omega t)
]

[
i_q = I_q + r\sin(\Omega t)
]

とします。

これは

**半径 r の円運動**

```id="slw5h3"
        iq
         ↑
     •       •
       ↘   ↗
         ●
       ↗   ↘
     •       •
           → id
```

---

# 2. dqリップルの相関を計算

リップル成分

[
\tilde i_d = r\cos(\Omega t)
]

[
\tilde i_q = r\sin(\Omega t)
]

積

[
\tilde i_d\tilde i_q
====================

r^2\cos(\Omega t)\sin(\Omega t)
]

恒等式

[
\cos x \sin x = \frac12\sin(2x)
]

より

[
\tilde i_d\tilde i_q
====================

\frac{r^2}{2}\sin(2\Omega t)
]

平均

[
\overline{\tilde i_d\tilde i_q}=0
]

---

# 3. つまり「円運動だけ」なら平均トルクは変わらない

円運動リップル

[
\begin{cases}
\tilde i_d = r\cos(\Omega t)\
\tilde i_q = r\sin(\Omega t)
\end{cases}
]

では

[
\overline{\tilde i_d\tilde i_q}=0
]

なので

**平均トルクは変わりません。**

---

# 4. 平均トルクが変わるケース

平均が変わるのは

[
\phi \neq \frac{\pi}{2}
]

つまり

[
\tilde i_d = r_d\cos(\Omega t)
]

[
\tilde i_q = r_q\cos(\Omega t+\phi)
]

で

[
\phi \neq 90^\circ
]

のとき。

すると

[
\overline{\tilde i_d\tilde i_q}
===============================

\frac12 r_d r_q \cos\phi
]

が残ります。

---

# 5. なぜ今回ズレるのか（重要）

あなたのケース

**3次零相 → dqで2ω**

このリップルは

* インバータ電圧誤差
* 電流制御器
* Ld/Lq非対称
* MTPA動作点

の影響で

**完全な円運動になりません。**

つまり

```id="e11mlg"
楕円運動
```

になります。

[
\tilde i_d = r_d\cos(2\omega t)
]

[
\tilde i_q = r_q\cos(2\omega t+\phi)
]

ここで

[
\phi \neq 90^\circ
]

となるため

[
\overline{\tilde i_d\tilde i_q} \neq 0
]

→ **平均トルクが変わる**

---

# 6. 「6次なら平均0」という直感の理由

多くのモータで

**6次リップルは**

[
\tilde i_d \propto \cos(6\omega t)
]

[
\tilde i_q \propto \sin(6\omega t)
]

になりやすく

**90°位相差**

になりやすい。

だから

[
\overline{\tilde i_d\tilde i_q}=0
]

になりやすいだけです。

---

# 7. まとめ

あなたの理解を整理すると

| ケース   | dqリップル形      | 平均トルク |
| ----- | ------------ | ----- |
| 円運動   | cos / sin    | 変化なし  |
| 楕円運動  | cos / cos(φ) | 変化あり  |
| φ=90° | 相関0          | 変化なし  |
| φ≠90° | 相関あり         | 変化    |

---

# 8. 今回の現象の本質

今回の

**3次零相 → dq2ω**

は

* dqベクトルが楕円運動
* ( \phi \neq 90° )

になるため

[
\overline{\tilde i_d\tilde i_q} \neq 0
]

→ **平均トルクが3%変化**

---

もしよければ次に、
あなたの現象で最も重要なポイント

**「なぜ3次零相リップルは dqで楕円運動になりやすいのか」**

（ここが今回のメカニズムの核心）

を説明します。
これが分かると今回の現象がほぼ完全に理解できます。
---
よい質問です。ここを整理すると、今回の現象（**抑制OFFで平均トルクが落ちる**）の理解がかなりクリアになります。

結論を先に言うと：

> **3次零相由来の成分は、Clarke→Park を通すと dq 空間で「円運動ではなく楕円運動」になりやすい。**
> その理由は
> **① 正相と逆相が同時に混ざる**
> **② モータの dq ダイナミクスが異方性（(L_d \neq L_q)）**
> の2つです。

順番に説明します。

---

# 1. dqで「円運動」になる条件

dqベクトルが円運動になる条件は

[
\tilde i_d = r\cos(\Omega t)
]

[
\tilde i_q = r\sin(\Omega t)
]

つまり

* 同じ振幅
* 位相差 (90^\circ)

です。

ベクトルとして

[
|\tilde{\mathbf i}| = r
]

一定。

これは

**単一の回転ベクトル**

のときだけ起こります。

---

# 2. αβ空間で単一回転ベクトルとは

例えば

[
i_\alpha = I\cos(\Omega t)
]

[
i_\beta = I\sin(\Omega t)
]

これは

**正相1個だけ**

です。

---

# 3. しかし3次零相が崩れると何が起きるか

あなたの系

* デュアルインバータ
* 電圧ばらつき
* 零相電流あり

このとき

3次電流は

[
i_a,i_b,i_c
]

で

**純零相にならない**

つまり

[
i^{(+)} + i^{(-)}
]

が発生します。

---

# 4. 正相＋逆相が同時に存在

αβで書くと

正相

[
\mathbf i^{(+)} =
\begin{bmatrix}
A\cos(3\omega t) \
A\sin(3\omega t)
\end{bmatrix}
]

逆相

[
\mathbf i^{(-)} =
\begin{bmatrix}
B\cos(3\omega t) \

* B\sin(3\omega t)
  \end{bmatrix}
  ]

合成

[
\mathbf i =
\begin{bmatrix}
(A+B)\cos(3\omega t) \
(A-B)\sin(3\omega t)
\end{bmatrix}
]

---

# 5. これが楕円

振幅が

[
\alpha : A+B
]

[
\beta : A-B
]

になるので

[
\left(\frac{i_\alpha}{A+B}\right)^2+
\left(\frac{i_\beta}{A-B}\right)^2=1
]

つまり

**楕円運動**

になります。

---

# 6. Park変換するとどうなるか

Park

[
\theta = \omega t
]

なので

3ω → 2ω に変換されます。

結果

[
i_d = (A+B)\cos(2\omega t)
]

[
i_q = (A-B)\sin(2\omega t)
]

---

# 7. ここで重要

振幅が違う

[
(A+B) \neq (A-B)
]

なので

[
\tilde i_d \neq \tilde i_q
]

つまり

**楕円**

になります。

---

# 8. なぜ3次零相は正相＋逆相になりやすいか

理由は

### ① デュアルインバータ電圧ばらつき

6相電圧

[
v_{a1},v_{b1},v_{c1},v_{a2},v_{b2},v_{c2}
]

が一致しない。

すると

零相電圧

[
v_0
]

だけでなく

**正相と逆相の3次**

が生成されます。

---

### ② モータ磁気異方性

IPMでは

[
L_d \neq L_q
]

なので

電流応答

[
i_d,i_q
]

のゲインが違う。

つまり

αβ空間の円運動が

**dqで歪む**

---

### ③ 電流制御器

PI制御は

[
d,q
]

で独立に動くため

リップル応答が異なる。

---

# 9. まとめ

**3次零相 → dq楕円運動**

の理由

1️⃣ 零相崩れ
→ 正相＋逆相

2️⃣ 正相＋逆相
→ αβ楕円

3️⃣ Park変換
→ dq楕円

---

# 10. ここが今回の核心

楕円になると

[
\tilde i_d \tilde i_q
]

の平均

[
\overline{\tilde i_d \tilde i_q}
]

が

**ゼロにならない**

可能性が高い。

つまり

[
\boxed{\text{平均トルクが変わる}}
]

---

もし希望あれば次に、

**今回の現象を最もシンプルに説明する1つの式**

[
\Delta T \approx \frac{3}{4}p(L_d-L_q) r_d r_q \cos\phi
]

を導出します。

これが

**今回の3%トルク低下の定量式**になります。
---
了解です。**「今回の現象を最もシンプルに説明する1つの式」**として

[
\Delta T \approx \frac{3}{4}p,(L_d-L_q),r_d r_q \cos\phi
]

（※あなたが貼った式は係数が前後して見えますが、導出すると **(3/4)** が出ます）
を、**Clarke/Parkの前段は置いておいて**「dqに同一周波数のリップルが乗った」ことから最短で導出します。

---

# 0) 前提（今回の“本質モデル”）

3次零相由来のリップルが dq に回り込み、dq電流が

[
\begin{aligned}
i_d(t)&=I_d + r_d\cos(\Omega t)\
i_q(t)&=I_q + r_q\cos(\Omega t+\phi)
\end{aligned}
]

と表せるとします。

* (I_d, I_q)：平均（DC）成分（ON/OFFで同じ）
* (r_d, r_q)：dqリップル振幅（OFFで大）
* (\Omega)：dq上で見えるリップル周波数
  （今回、3次→dqでは典型的に (\Omega=2\omega)）
* (\phi)：dqリップルの位相差（＝相関を決める）

---

# 1) トルク式（IPMSMの標準式）

電磁トルクを

[
T(t)=\frac{3}{2}p\Big(\psi_f i_q(t) + (L_d-L_q),i_d(t)i_q(t)\Big)
]

とします。

今回の「平均トルク差」を最も単純に説明するのは **第2項（リラクタンストルク）**です。
（第1項は平均 (I_q) が同じなら平均は基本同じになりやすい）

---

# 2) 平均トルク差の“発生源”は (\overline{i_d i_q})

ON/OFFで (I_d,I_q) が同じなら、差が出るのは

[
\overline{i_d i_q}
]

の違いです。

まず積を展開します：

[
\begin{aligned}
i_d i_q
&=\left(I_d + r_d\cos\Omega t\right)\left(I_q + r_q\cos(\Omega t+\phi)\right)\
&=I_d I_q

* I_d r_q\cos(\Omega t+\phi)
* I_q r_d\cos(\Omega t)
* r_d r_q \cos(\Omega t)\cos(\Omega t+\phi)
  \end{aligned}
  ]

時間平均を取ります（(\overline{\cos}=0) を使う）：

* 第2項、第3項は平均0
* 問題は第4項だけ

---

# 3) 重要ステップ：(\overline{\cos(\Omega t)\cos(\Omega t+\phi)})

恒等式

[
\cos A\cos B=\frac12\left[\cos(A-B)+\cos(A+B)\right]
]

により

[
\cos(\Omega t)\cos(\Omega t+\phi)
=\frac12\left[\cos\phi+\cos(2\Omega t+\phi)\right]
]

平均を取ると (\overline{\cos(2\Omega t+\phi)}=0) なので

[
\overline{\cos(\Omega t)\cos(\Omega t+\phi)}
=\frac12\cos\phi
]

よって

[
\overline{i_d i_q}
= I_d I_q + \frac12 r_d r_q \cos\phi
]

---

# 4) 平均トルクの差分（ΔT）の導出

平均トルクは

[
\overline{T}=\frac{3}{2}p\Big(\psi_f \overline{i_q} + (L_d-L_q)\overline{i_d i_q}\Big)
]

ここで (\overline{i_q}=I_q) なので、ON/OFFで (I_q) が同じなら PM項は差が出ません。

差が出るのはリラクタンストルク項：

[
\Delta \overline{T}
= \frac{3}{2}p (L_d-L_q)\Delta\overline{i_d i_q}
]

ON（抑制あり）でリップルが十分小さいとして (r_d=r_q\approx 0) なら

[
\Delta\overline{i_d i_q}\approx \frac12 r_d r_q \cos\phi
]

よって

[
\boxed{
\Delta T \approx \frac{3}{2}p (L_d-L_q)\cdot \frac12 r_d r_q \cos\phi
}
]

整理して

[
\boxed{
\Delta T \approx \frac{3}{4}p (L_d-L_q), r_d r_q \cos\phi
}
]

これで導出完了です。

---

# 5) この式の“読み方”（現象との対応）

* (r_d,r_q) が増える（抑制OFF）ほど、平均トルク差が出やすい
* (\cos\phi) の符号で増減が決まる

  * (\phi\approx 90^\circ) なら差はほぼ0
  * (\phi\approx 0^\circ) なら最大
* IPMで (L_d\neq L_q) だからこの項が効く（SPMなら弱い）

---

# 6) 注意：これは「平均トルク差の最小モデル」

この式は **“電磁トルクの平均がズレる最小要因”**を抜き出したものです。
実際の「軸トルク3%低下」には加えて

* RMS増による銅損増（軸トルク低下に直結）
* 電圧飽和で (I_q) 有効成分が少し削られる
* (L_d,L_q,\psi_f) の電流依存（二次効果）
* トルクセンサ帯域・フィルタ

も乗り得ます。

---

必要なら、次の一手として（実務向けに）
ログから (r_d,r_q,\phi) を推定して、この式が **3%** を説明できるか（オーダ感）をチェックする手順も出せます。
