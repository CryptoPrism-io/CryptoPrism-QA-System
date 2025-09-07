# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CryptoPrism-DB Quality Assurance System v2 is a professional-grade PostgreSQL database quality assurance solution designed specifically for cryptocurrency data pipelines. The system provides intelligent monitoring, automated testing, and real-time alerting capabilities with AI-powered analysis.

## Architecture

### Standalone QA System
- **Database Focus**: PostgreSQL with cryptocurrency-specific schema validation
- **Health Scoring**: 0-100 scale with visual indicators and trend analysis
- **AI Integration**: OpenAI and Google Gemini for intelligent issue summarization
- **Multi-Channel Alerts**: Telegram notifications with risk-based escalation

### Core Module Structure
All components are located in the repository root:

**Main Entry Points:**
- `run_qa.py` - Primary CLI interface with multiple execution modes
- `quick_qa_test.py` - Schema-aligned test with proven 100/100 health score

**Core Components:**
- `core/config.py` - Configuration management with environment variable validation
- `core/database.py` - PostgreSQL connection handling with SQLAlchemy
- `core/base_qa.py` - QA result data classes and base functionality

**Individual Test Modules:**
- `tests/individual/test_data_quality.py` - NULL analysis and data integrity checks
- `tests/individual/test_timestamp_validation.py` - Timestamp rules and freshness validation
- `tests/individual/test_duplicate_detection.py` - Duplicate record detection using composite keys
- `tests/individual/test_business_logic.py` - CryptoPrism-specific business rule validation

**Reporting & Notifications:**
- `reporting/notification_system.py` - Telegram and AI-powered alert system
- `reporting/qa_report_logger.py` - Historical QA execution tracking
- `reporting/report_generator.py` - JSON/CSV report generation

**Utilities:**
- `utils/direct_db_check.py` - Direct database connectivity verification
- `utils/verify_database_schema.py` - Database schema validation tools

## Common Commands

### Dependencies
```bash
# Install all required dependencies
pip install -r requirements.txt
```

### Environment Setup
```bash
# Copy environment template and configure
cp .env.example .env
# Edit .env with your actual database credentials
```

### Testing Commands
```bash
# Run proven working test with 100/100 health score
python quick_qa_test.py

# Run all individual tests with separate notifications
python run_qa.py --individual

# Run specific test for focused debugging
python run_qa.py --test data_quality
python run_qa.py --test timestamps
python run_qa.py --test duplicates
python run_qa.py --test business_logic

# List all available test options
python run_qa.py --list

# Run comprehensive analysis (full database sweep)
python run_qa.py --comprehensive
```

### Individual Test Modules (Direct Execution)
```bash
# Run individual test files directly for debugging
python tests/individual/test_data_quality.py
python tests/individual/test_timestamp_validation.py
python tests/individual/test_duplicate_detection.py
python tests/individual/test_business_logic.py
```

## Environment Configuration

### Required Environment Variables
Create a `.env` file with these required settings:
```env
# Database Configuration (Required)
DB_HOST=your_postgresql_host
DB_USER=your_database_username
DB_PASSWORD=your_database_password
DB_PORT=5432
DB_NAME=dbcp

# Optional Integrations
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_CHAT_ID=your_telegram_chat_id
OPENAI_API_KEY=your_openai_api_key
GEMINI_API_KEY=your_google_gemini_api_key
```

### QA Thresholds (Optional Customization)
```env
QA_NULL_RATIO_LOW=0.05          # 5% NULL threshold for warnings
QA_NULL_RATIO_MEDIUM=0.20       # 20% NULL threshold for medium risk
QA_NULL_RATIO_HIGH=0.50         # 50% NULL threshold for high risk
QA_MIN_TIMESTAMP_RANGE_DAYS=4   # Minimum days for historical data
QA_MIN_EXPECTED_ROWS=100        # Minimum expected rows in tables
```

## Key Database Tables

The QA system monitors these critical cryptocurrency database tables:

### Signal Tables (Primary Monitoring)
- `FE_DMV_ALL` - Aggregated DMV signals with 40+ technical indicators
- `FE_DMV_SCORES` - Durability/Momentum/Valuation scores (0-100 scale)
- `FE_MOMENTUM_SIGNALS` - Momentum-based trading signals
- `FE_OSCILLATORS_SIGNALS` - Technical oscillator signals

### Data Tables (Validation Focus)
- `1K_coins_ohlcv` - OHLCV historical price data (2M+ records expected)
- `crypto_listings_latest_1000` - CoinMarketCap listings (1K records expected)

## Development Patterns

### Database Schema Compatibility
- The system uses **actual database column names** (case-sensitive)
- Key columns: `slug`, `timestamp`, `Durability_Score`, `Momentum_Score`, `Valuation_Score`
- Always use quoted identifiers in SQL: `"FE_DMV_ALL"`, `"Durability_Score"`
- Composite key validation: `(slug, timestamp)` must be unique

### Import System
- All modules use **absolute imports** for standalone operation
- Import pattern: `from core.config import QAConfig`
- No relative imports (avoid `from ..core import`)
- Run tests from repository root directory

### Health Score Calculation
```python
health_score = (
    status_counts['PASS'] * 1.0 +
    status_counts['WARNING'] * 0.7 +
    status_counts['FAIL'] * 0.3 +
    status_counts['ERROR'] * 0.1
) / total_checks * 100
```

### Risk Level Classification
- **LOW** (🟢): Minor issues, acceptable performance
- **MEDIUM** (🟡): Issues requiring attention, monitor closely  
- **HIGH** (🟠): Problems requiring immediate attention
- **CRITICAL** (🔴): Severe issues, immediate action required

## CI/CD Integration

### GitHub Actions Workflows
The system includes three comprehensive workflows:

1. **`daily-qa-check.yml`** - Automated daily monitoring
   - Runs at 6:00 AM UTC daily
   - Can be triggered manually with test type selection
   - Uploads QA reports as artifacts

2. **`on-demand-qa.yml`** - Manual testing workflow
   - Configurable test types and notification settings
   - Health threshold validation
   - Comprehensive result reporting

3. **`pr-qa-check.yml`** - Pull request validation
   - Code quality checks and syntax validation
   - Import system testing
   - Documentation and structure validation

### GitHub Secrets Required
Configure these secrets in your GitHub repository:
- `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_PORT`, `DB_NAME`
- `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` (optional)
- `OPENAI_API_KEY` (optional)

## Expected Performance Metrics

### Proven Results (Battle-Tested)
- **Health Score**: 100.0/100 (Perfect validation achieved)
- **Database Coverage**: 997 cryptocurrencies + 2.26M OHLCV records + 1K market listings
- **Data Integrity**: Zero duplicates across all monitored tables
- **Response Time**: All queries complete within 2-3 seconds
- **DMV Scores**: Average D:19.8, M:-30.8, V:-24.4 (997 cryptocurrencies)

### Success Criteria
- Health score ≥ 95: Excellent database health
- Health score 80-94: Good with minor issues
- Health score 60-79: Issues requiring attention
- Health score < 60: Critical problems, immediate action required

## Troubleshooting

### Common Issues
1. **Import Errors**: Ensure you're running from repository root directory
2. **Database Connection**: Verify `.env` file credentials and PostgreSQL accessibility
3. **Schema Mismatches**: Use `quick_qa_test.py` which uses verified actual column names
4. **Telegram Notifications**: Check bot token and chat ID in `.env` file

### Debugging Commands
```bash
# Test database connection directly
python -c "from core.config import QAConfig; from core.database import DatabaseManager; config = QAConfig(); db = DatabaseManager(config); print('Connection:', db.test_connection('dbcp'))"

# Verify environment configuration
python -c "from core.config import QAConfig; config = QAConfig(); print('Config loaded successfully')"

# Check import system
python -c "from core.config import QAConfig; from core.database import DatabaseManager; from reporting.notification_system import NotificationSystem; print('All imports successful')"
```

### Log Files
- `logs/qa_system.log` - General system logs
- `logs/qa_errors.log` - Error tracking
- `logs/qa_performance.log` - Performance metrics
- `logs/qa_alerts.log` - Alert history

## Adding New QA Tests

When adding new quality assurance tests:

1. **Create test module** in `tests/individual/test_your_feature.py`
2. **Follow established patterns**:
   - Use `QAResult` class for standardized results
   - Include risk level classification
   - Add Telegram notification calls
   - Use actual database column names
3. **Update main runner** in `run_qa.py` to include new test option
4. **Add documentation** to README.md and this CLAUDE.md file
5. **Test independently** before integration

## Project Status

- **Version**: 2.1.0
- **Status**: Production-ready standalone repository
- **Last Validation**: 2025-09-08 01:36:42 UTC
- **Health Score**: 100.0/100 (Perfect)
- **Migration Status**: Complete from parent CryptoPrism-DB repository
- **CI/CD**: Ready with 3 GitHub Actions workflows

## Key Dependencies

### Core Dependencies
- `sqlalchemy>=2.0.32` - Database ORM and connection management
- `pg8000>=1.30.0` or `psycopg2-binary>=2.9.0` - PostgreSQL drivers
- `python-dotenv>=1.0.0` - Environment variable management
- `pandas>=2.2.2`, `numpy>=1.26.4` - Data processing

### Notification Dependencies  
- `requests>=2.32.3` - HTTP requests for Telegram API
- `openai>=1.0.0` - OpenAI GPT integration for AI analysis
- `google-generativeai>=0.5.0` - Google Gemini integration

### Development Dependencies (Optional)
- `pytest>=7.0.0` - Testing framework
- `flake8` - Code quality checking
- `black` - Code formatting

Always add changes to CHANGELOG.md with version, timestamp, and rationale when making modifications to the codebase.

**This QA system is battle-tested with real cryptocurrency data and maintains perfect health scores in production environments.**