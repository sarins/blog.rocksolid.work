FROM continuumio/miniconda3:24.4.0-0

EXPOSE 9900
WORKDIR /opt
# RUN /bin/bash
#RUN conda install -c conda-forge jupyter -y --quiet
#RUN conda install -c conda-forge prophet -y --quiet
RUN conda install -c conda-forge jupyter -y
RUN conda install -c conda-forge prophet -y
# RUN conda install -c conda-forge plotly -y

RUN mkdir -p /opt/notebooks
CMD ["jupyter", "notebook", "--notebook-dir=/opt/notebooks", "--ip='*'", "--port=9900", "--no-browser", "--allow-root"]