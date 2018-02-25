# Batched Kronecker Product Test
cupyで性能差が見られなかったので生CUDAで書いてみようと  
正しくはBatchedではなくStreamedだけど
[StreamedとBatchedのgemm比較](https://devblogs.nvidia.com/cublas-strided-batched-matrix-multiply/)

## ファイル
- kp.py : BatchedKroneckerProductのPythonのコード 
- main.cu : BatchedKroneckerProductの生CUDAのコード

## Cupyの問題点

- matmulで呼んでいるcublasSgemmの転置引数がCUBLAS\_OP\_Nで固定されている  
→ 転置した行列の積を計算したければtranspose()を呼ばなくてはいけない  
→ cupy.copyが呼ばれる(無駄)

- cudaMemsetがいたるところで呼ばれている  
→ 重くはないけど無駄(gemm計算の1.5%程度の時間)

- matmulがstream指定を無視している気がする  
→ nvprofで見るとgemm計算でのstream idが変化していない  
→ Batchedできていない

### 解決法
- cublasSgemmを直接呼ぶ→転置の問題は解決
- cupy使わない(chainerもできれば使わない)→面倒
- ボトルネックでなければ無視→願う


## 実験
生CUDAで

### 環境
- GF GTX 1080

### 結果

### 高速化率
- バッチサイズ : 30
- 計算回数 : 100
- 行列サイズ : 変化

![グラフ](./speedup.png)

(y=1のgridだけ太くしたかった)


行列のサイズがある程度の大きさまではバッチ処理の方が速くなるみたい
