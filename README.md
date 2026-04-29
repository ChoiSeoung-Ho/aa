graph TD
    %% Main Pipeline
    subgraph Main_Pipeline ["Main Pipeline"]
        direction LR
        M_In([Input]) --> M_Eff[EfficientNetV2<br>Backbone]
        M_Eff --> M_Reshape[Reshape<br>1D -> 4D]
        M_Reshape --> M_CBAM[CBAM]
        M_CBAM --> M_CICA["CICA Module<br>(SAFE)"]
        M_CICA --> M_GAP[GAP + Flatten]
        M_GAP --> M_Class[Classifier]
        M_Class --> M_Out([Output])
    end

    %% (a) CBAM Block
    subgraph Block_A ["(a) CBAM Block"]
        direction TB
        A_In([Input]) --> A_CA[Channel Attention<br>Module]
        A_In --> A_Mul1((X))
        A_CA --> A_LRN1[LRN]
        A_LRN1 --> A_Mul1
        
        A_Mul1 --> A_SA[Spatial Attention<br>Module]
        A_Mul1 --> A_Mul2((X))
        A_SA --> A_Mul2
        A_Mul2 --> A_LRN2[LRN]
        A_LRN2 --> A_Out([Output])
    end

    %% (b) CICA Module (SAFE)
    subgraph Block_B ["(b) CICA Module (SAFE)"]
        direction TB
        B_In([Input]) --> B_DW[DepthwiseConv 3x3]
        B_In --> B_Skip[Conv 1x1<br>Skip Connection]
        
        B_DW --> B_PW[PointwiseConv 1x1]
        B_PW --> B_CELU1[CELU]

        B_CELU1 --> B_D1[Dilated Conv 3x3, d=1<br>+ CELU]
        B_CELU1 --> B_D2[Dilated Conv 3x3, d=2<br>+ CELU]
        B_CELU1 --> B_D4[Dilated Conv 3x3, d=4<br>+ CELU]

        B_D1 --> B_M1(("X<br>(d1xd2)"))
        B_D2 --> B_M1
        
        B_D2 --> B_M2(("X<br>(d2xd4)"))
        B_D4 --> B_M2
        
        B_D1 --> B_M3(("X<br>(d1xd4)"))
        B_D4 --> B_M3

        B_M1 --> B_Concat[Concatenate]
        B_M2 --> B_Concat
        B_M3 --> B_Concat

        B_Concat --> B_GAP[GlobalAveragePooling2D]
        B_Concat --> B_AttMul((X))

        B_GAP --> B_FCR[FC Reduction]
        B_FCR --> B_CELU2[CELU]
        B_CELU2 --> B_FCE[FC Expansion]
        B_FCE --> B_CELU3[CELU]

        B_CELU3 --> B_AttMul

        B_AttMul --> B_Fusion[Fusion Conv 3x3<br>+ CELU]
        
        B_Fusion --> B_Add((+))
        B_Skip --> B_Add
        B_Add --> B_Out([Output])
    end

    %% (c) Classifier
    subgraph Block_C ["(c) Classifier"]
        direction TB
        C_In([Input]) --> C_LN[LayerNorm]
        C_LN --> C_Lin1[Linear<br>Hidden Dim]
        C_Lin1 --> C_GELU[GELU]
        C_GELU --> C_Drop[Dropout<br>p=0.1]
        C_Drop --> C_Lin2[Linear<br>Num Labels]
        C_Lin2 --> C_Out([Output])
    end
