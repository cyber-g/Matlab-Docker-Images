FROM mathworks/matlab-deep-learning:r2024a

USER root
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates wget \
    && rm -rf /var/lib/apt/lists/*

USER matlab
WORKDIR /tmp
RUN wget -q https://www.mathworks.com/mpm/glnxa64/mpm -O mpm \
    && chmod +x mpm \
    && MATLAB_ROOT="$(dirname "$(dirname "$(readlink -f "$(command -v matlab)")")")" \
    && sudo HOME="${HOME}" ./mpm install \
        --release=R2024a \
        --destination="${MATLAB_ROOT}" \
        --products Communications_Toolbox DSP_System_Toolbox Signal_Processing_Toolbox Simulink \
    && sudo rm -f mpm /tmp/mathworks_root.log

WORKDIR /home/matlab
