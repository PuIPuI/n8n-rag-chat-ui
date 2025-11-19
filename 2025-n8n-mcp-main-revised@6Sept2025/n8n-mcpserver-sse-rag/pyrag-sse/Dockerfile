# Multi-stage build for smaller final image
FROM python:3.11-slim as builder

# Install build dependencies
RUN apt-get update && apt-get install -y \
 gcc \
 g++ \
 && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements first for better caching
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir --user -r requirements.txt



# ==================== FINAL STAGE ====================
FROM python:3.11-slim

# ✅ รับค่า QP_DOWNLOAD_URL จากภายนอกตอน build
ARG QP_DOWNLOAD_URL

# Install runtime dependencies including telnet, wget, curl, unzip
RUN apt-get update && apt-get install -y \
 libgomp1 \
 telnet \
 wget \
 curl \
 unzip \
 && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -u 1000 mcpuser

# Create documents directory
RUN mkdir -p /home/mcpuser/documents && chown mcpuser:mcpuser /home/mcpuser/documents

# ✅ โหลด QP ZIP จาก QP_DOWNLOAD_URL แล้วแตกไฟล์เข้า /home/mcpuser/documents
RUN echo "Downloading QP docs from: ${QP_DOWNLOAD_URL}" \
 && if [ -z "$QP_DOWNLOAD_URL" ]; then echo "ERROR: QP_DOWNLOAD_URL is not set"; exit 1; fi \
 && curl -L "${QP_DOWNLOAD_URL}" -o /tmp/QP-ALL.zip \
 && unzip /tmp/QP-ALL.zip -d /home/mcpuser/documents \
 && rm /tmp/QP-ALL.zip \
 && chown -R mcpuser:mcpuser /home/mcpuser/documents

# Set working directory
WORKDIR /app

# Copy Python dependencies from builder
COPY --from=builder /root/.local /home/mcpuser/.local

# Copy application code
COPY --chown=mcpuser:mcpuser . .

# ❌ ลบส่วนนี้ออกแล้ว เพราะเราไม่อยาก COPY *.md จาก git เข้า image อีก
# COPY --chown=mcpuser:mcpuser *.md /home/mcpuser/documents/

# Switch to non-root user
USER mcpuser

# Add local bin to PATH
ENV PATH=/home/mcpuser/.local/bin:$PATH

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
 CMD python -c "import httpx; httpx.get('http://localhost:8000/health').raise_for_status()"

# Run the application
CMD ["python", "main.py"]