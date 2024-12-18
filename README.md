# D3D12ExecuteIndirect
A repository to demonstrate a possible bug in AMD's DX12 drivers related to Shader Model 6.6's direct heap indexing, specifically in compute shaders. This is a slightly modified version of Microsoft's D3D12ExecuteIndirect sample, changed to demonstrate this bug.

Expected: The program should function like the ExecuteIndirect sample here: https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12ExecuteIndirect

What actually happens:

On an Nvidia GPU (3090 ti, 1660 super, 1070): The program functions as expected

On an RX 7000 series GPU (7800XT): Nothing is drawn while culling is enabled. Looking at the dispatches in PIX, it is clear that something is wrong. PIX doesn't quite get it, but all of the reads from the SRV buffer are returning zeros, and all of the writes to the output buffer are silently failing. If the output buffer is attached in a root descriptor table, writes will no longer fail, so its counter will increase, but it will still be full of zeros singe the input buffer is broken.

On an RX 6000 series GPU (6800XT): The D3D12 device is removed after the dispatch, and the program crashes.

On an Polaris seris GPU (RX 580): Same as RX 7000 series

What seems to be causing this: It seems like the SM6.6 directly indexed heap feature is not working correctly in Polaris, RX6000 and RX7000 series drivers. I'm not sure about RDNA 1, since I don't have one of those. This only happens in compute shaders- in mesh, vertex, and pixel shaders, it seems to work fine.