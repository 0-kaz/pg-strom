@ja:#PG-Strom v3.0リリース
@en:#PG-Strom v3.0 Release

<div style="text-align: right;">PG-Strom Development Team (12-May-2021)</div>

@ja:##概要
@en:##Overview

@ja{
PG-Strom v3.0における主要な機能強化は以下の通りです。

- いくつかのPostGIS関数がGPUで実行可能となりました。
- GiSTインデックスを使用したGpuJoinが可能となりました。
- GPUキャッシュ機能が実装されました。
    - これは過去のバージョンで実装されていた同名の機能とは別物です。
- NVIDIA GPUDirect Storageに対応しました。（実験的）
- ユーザ定義のGPUデータ型/関数/演算子に対応しました。
}

@en{
Major enhancement in PG-Strom v3.0 includes:

- Some PostGIS functions can now be executed on GPUs.
- GpuJoin using GiST index is now possible.
- GPU Cache mechanism has been implemented.
    - This is different from the feature of the same name that was implemented in past versions.
- NVIDIA GPUDirect Storage is now supported. (Experimental)
- User-defined GPU data types/functions/operators are now supported.
}


@ja:##動作環境
@en:##Prerequisites

@ja{
- PostgreSQL v11, v12, v13
- CUDA Toolkit 11.2 以降
- CUDA ToolkitのサポートするLinuxディストリビューション
- Intel x86 64bit アーキテクチャ(x86_64)
- NVIDIA GPU CC 6.0 以降 (Pascal以降)
}
@en{
- PostgreSQL v11, v12, v13
- CUDA Toolkit 11.2 or later
- Linux distributions supported by CUDA Toolkit
- Intel x86 64bit architecture (x86_64)
- NVIDIA GPU CC 6.0 or later (Pascal or newer)
}

@ja:##新機能
@en:##New Features

@ja:###GPU版PostGIS対応
@en:###GPU-PostGIS support

@ja{
いくつかのPostGIS関数にGPU版を実装しました。
条件句でこれらのPostGIS関数が使用されている場合、PG-StromはGPU側でこれを実行するようGPUプログラムを自動生成します。
}
@en{
We have implemented GPU versions of several PostGIS functions. 
If any of these PostGIS functions are used in a conditional clause, PG-Strom will automatically generate a GPU program to execute it on the GPU.
}

@ja:###GiSTインデックス対応
@en:###GiST-index support

@ja{
ジオメトリ型の列にGiSTインデックスが設定されており、かつ結合条件がインデックスによる絞り込みに対応している場合、
GpuJoinはGPU側へGiSTインデックスをロードし、これを用いてテーブル同士のJOINを実行します。

GiSTインデックスの探索はGPUで並列に実行されるため、大幅なパフォーマンスの改善が期待できます。
}
@en{
If a column whose data type is geometry has a GiST index and the table can be filtered using the index, GpuJoin will load the GiST index on the GPU and use it to join the tables.

Since the GiST index search is executed in parallel on the GPU, significant performance improvements can be expected.
}

@ja:###GPUキャッシュ機能
@en:###GPU Cache mechanism

@ja{
GPUデバイスメモリ上に予め領域を確保しておき、対象となるテーブルの複製を保持しておく機能です。
比較的小規模のデータサイズ（～10GB程度）で、更新頻度の高いデータに対して分析/検索系のクエリを効率よく実行するために設計されました。

分析/検索系クエリの実行時には、GPU上のキャッシュを参照する事で、テーブルからデータを読み出す事なくGPUでSQLワークロードを処理する事が可能です。
}
@en{
GPU Cache mechanism can store a copy of the target table in a pre-allocated area on the GPU device memory.
It was designed for efficient execution of analytical/search queries on frequently updated data with relatively small data size (~10GB).

The GPU can process SQL workloads by referring to GPU Cache instead of loading data from tables when executing analytical/search queries.
}

@ja:###NVIDIA GPUDirect Storage (実験的対応)
@en:###NVIDIA GPUDirect Storage (experimental)

@ja{
[GPUダイレクトSQL](../ssd2gpu)用のドライバとして、従来の nvme_strom カーネルモジュールに加えて、
NVIDIAが開発を進めている[GPUDirect Storage](https://developer.nvidia.com/blog/gpudirect-storage/)にも対応しました。

これにより、従来からのNVME-SSDやNVME-oFストレージに加えて、SDS(Software Defined Storage)デバイスからのGPUダイレクトSQLが可能となります。

現在、GPUDirect Storage用のドライバはベータ版のため実験的対応となっています。
}
@en{
[GPUDirect Storage](https://developer.nvidia.com/blog/gpudirect-storage/) developed by NVIDIA is now available as a driver for [GPU Direct SQL](../ssd2gpu), in addition to the existing nvme_strom kernel module.

GPU Direct SQL will now be available on SDS (Software Defined Storage) devices in addition to conventional NVME-SSD and NVME-oF storage.

Currently, the driver for GPUDirect Storage is a beta version and experimental support is available.
}

@ja:###ユーザ定義のGPUデータ型/関数
@en:###User-defined GPU datatype/functions

@ja{
ユーザ定義のGPUデータ型/関数を追加するためのAPIを新たに提供します。
これにより、PG-Strom自体には手を加えることなく、ニッチな用途のデータ型やそれを処理するためのSQL関数をユーザが独自に定義、実装する事が可能となりました。
}

@en{
A new API is provided to add user-defined GPU data types/functions. This allows users to define and implement their own niche data types and SQL functions to process them, without modifying PG-Strom itself.
}

@ja:##その他の変更
@en:##Other updates

@ja{
- 独自に `int1` (8bit整数) データ型、および関連する演算子に対応しました。
- `pg2arrow`に`--inner-join`および`--outer-join`オプションを追加しました。PostgreSQLの列数制限を越えた数の列を持つApache Arrowファイルを生成できるようになります。
- マルチGPU環境では、GPUごとに専用のGPU Memory Keeperバックグラウンドワーカーが立ち上がるようになりました。
- PostgreSQL v13.x に対応しました。
- CUDA 11.2 および Ampere世代のGPUに対応しました。
- ScaleFlux社のComputational Storage製品CSD2000シリーズに対応しました。
    - NVIDIA GPUDirect Storageの使用時（`nvme_strom`カーネルモジュールは対応していません）

}
@en{
- Unique int1 (8-bit integer) data type and related operators are now supported.
- `--inner-join` and `--outer-join` options are now available for `pg2arrow`. Apache Arrow files having more columns than the limit of PostgreSQL can now be generated.
- In a multi-GPU environment, the GPU Memory Keeper background worker will now be launched for each GPU.
- PostgreSQL v13.x is now supported.
- CUDA 11.2 and Ampere generation GPUs are now supported.
- ScaleFlux's Computational Storage product CSD2000 series is now supported.
    - NVIDIA GPUDirect Storage only (nvme_strom kernel module is not supported)
}

@ja:##廃止された機能
@en:##Deprecated Features

@ja{
- PostgreSQL v10.x 系列のサポートが廃止されました。
- Pythonスクリプトとのデータ連携機能（PyStrom）が廃止されました。
}
@en{
- Support for PostgreSQL v10.x has been discontinued.
- The feature to link data with Python scripts (PyStrom) has been discontinued.
}





