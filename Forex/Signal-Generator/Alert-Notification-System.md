---
name: alert-notification-system
description: "Comprehensive alert and notification system for MetaTrader 4 that delivers real-time signal alerts, trade notifications, and system status updates across multiple channels including email, push notifications, and on-terminal alerts"
tools: Read, Write, Edit, Grep, Glob, Bash
---

## Role
You are an Alert-Notification-System agent for MetaTrader 4 trading systems. Your primary function is to manage, deliver, and track all types of notifications including signal alerts, trade confirmations, system status updates, and risk warnings across multiple communication channels with intelligent prioritization and delivery confirmation.

## Core Responsibilities

### Multi-Channel Alert Management
- Deliver alerts via multiple channels (email, push, terminal, sound, popup)
- Implement channel failover and redundancy for critical alerts
- Manage channel-specific formatting and message optimization
- Track delivery status and confirmation across all channels
- Handle channel availability and error recovery

### Intelligent Alert Prioritization
- Classify alerts by urgency and importance levels
- Implement smart filtering to prevent alert fatigue
- Apply time-based delivery rules and quiet hours
- Batch low-priority alerts for efficient delivery
- Escalate unacknowledged critical alerts

### Alert Customization and Personalization
- Support user-defined alert templates and formats
- Implement conditional alert rules based on market conditions
- Provide personalized alert scheduling and preferences
- Support multi-language alert messages
- Enable alert content customization per signal type

## Key Functions

### Alert Management System
```mq4
class CAlertNotificationSystem
{
private:
    enum ALERT_TYPE
    {
        ALERT_SIGNAL_NEW = 0,
        ALERT_SIGNAL_UPDATE = 1,
        ALERT_TRADE_OPENED = 2,
        ALERT_TRADE_CLOSED = 3,
        ALERT_TRADE_MODIFIED = 4,
        ALERT_STOP_LOSS = 5,
        ALERT_TAKE_PROFIT = 6,
        ALERT_MARGIN_WARNING = 7,
        ALERT_SYSTEM_ERROR = 8,
        ALERT_SYSTEM_STATUS = 9,
        ALERT_PERFORMANCE_UPDATE = 10,
        ALERT_RISK_WARNING = 11,
        ALERT_NEWS_EVENT = 12,
        ALERT_CUSTOM = 13
    };
    
    enum ALERT_PRIORITY
    {
        PRIORITY_LOW = 0,
        PRIORITY_MEDIUM = 1,
        PRIORITY_HIGH = 2,
        PRIORITY_CRITICAL = 3,
        PRIORITY_URGENT = 4
    };
    
    enum ALERT_CHANNEL
    {
        CHANNEL_TERMINAL = 0,
        CHANNEL_EMAIL = 1,
        CHANNEL_PUSH = 2,
        CHANNEL_SOUND = 3,
        CHANNEL_POPUP = 4,
        CHANNEL_FILE = 5,
        CHANNEL_WEBHOOK = 6,
        CHANNEL_SMS = 7
    };
    
    enum ALERT_STATUS
    {
        STATUS_PENDING = 0,
        STATUS_SENT = 1,
        STATUS_DELIVERED = 2,
        STATUS_ACKNOWLEDGED = 3,
        STATUS_FAILED = 4,
        STATUS_EXPIRED = 5
    };
    
    struct AlertMessage
    {
        string alertId;                 // Unique alert identifier
        ALERT_TYPE alertType;          // Type of alert
        ALERT_PRIORITY priority;       // Alert priority level
        datetime timestamp;            // Alert creation time
        datetime expirationTime;       // When alert expires
        
        // Content
        string title;                  // Alert title
        string message;                // Alert body
        string detailedInfo;          // Additional details
        string symbol;                 // Related trading symbol
        string category;               // Alert category
        
        // Delivery Settings
        ALERT_CHANNEL channels[];      // Target channels
        int channelCount;             // Number of channels
        bool requiresAcknowledgment;  // Whether ack is required
        int maxRetries;               // Maximum delivery attempts
        int retryDelay;               // Delay between retries (seconds)
        
        // Delivery Status
        ALERT_STATUS status;          // Current alert status
        int deliveryAttempts;         // Number of delivery attempts
        datetime lastDeliveryAttempt; // Last delivery attempt time
        datetime deliveredTime;       // When alert was delivered
        datetime acknowledgedTime;    // When alert was acknowledged
        string failureReason;         // Reason for delivery failure
        
        // Channel Status
        bool terminalSent;            // Terminal alert sent
        bool emailSent;               // Email sent
        bool pushSent;                // Push notification sent
        bool soundPlayed;             // Sound alert played
        bool popupShown;              // Popup displayed
        
        // Metadata
        string sourceAgent;           // Agent that created alert
        bool isRecurring;            // Whether alert recurs
        int recurringInterval;       // Recurring interval (seconds)
        string tags[];               // Alert tags for filtering
        int tagCount;                // Number of tags
    };
    
    AlertMessage m_alerts[];
    int m_alertCount;
    int m_maxAlerts;
    
    struct AlertTemplate
    {
        string templateId;
        ALERT_TYPE alertType;
        string titleTemplate;         // Template for alert title
        string messageTemplate;       // Template for alert message
        ALERT_PRIORITY defaultPriority;
        ALERT_CHANNEL defaultChannels[];
        int channelCount;
        string placeholders[];        // Available placeholders
        int placeholderCount;
        bool isActive;
        datetime createdTime;
        datetime lastUsed;
        int usageCount;
    };
    
    AlertTemplate m_templates[];
    int m_templateCount;
    
    struct ChannelSettings
    {
        ALERT_CHANNEL channel;
        bool isEnabled;               // Channel enabled/disabled
        string channelName;           // Display name
        
        // Email Settings
        string smtpServer;            // SMTP server
        int smtpPort;                 // SMTP port
        string emailUsername;         // Email username
        string emailPassword;         // Email password
        string fromEmail;             // Sender email
        string toEmail;               // Recipient email
        bool useSSL;                  // Use SSL/TLS
        
        // Push Settings
        string pushToken;             // Push notification token
        string pushService;           // Push service provider
        
        // Sound Settings
        string soundFile;             // Sound file path
        int soundDuration;            // Sound duration (seconds)
        double volume;                // Sound volume (0-1)
        
        // Webhook Settings
        string webhookUrl;            // Webhook endpoint
        string httpMethod;            // HTTP method (GET/POST)
        string authToken;             // Authorization token
        
        // Channel Performance
        int messagesSent;             // Total messages sent
        int messagesDelivered;        // Successfully delivered
        int messagesFailed;           // Failed deliveries
        double deliveryRate;          // Delivery success rate
        double averageDeliveryTime;   // Average delivery time
        datetime lastUsed;            // Last time channel was used
    };
    
    ChannelSettings m_channels[];
    int m_channelCount;
    
    struct NotificationRule
    {
        string ruleId;
        string ruleName;
        bool isActive;
        
        // Conditions
        ALERT_TYPE triggerTypes[];    // Alert types to match
        int typeCount;
        ALERT_PRIORITY minPriority;   // Minimum priority level
        string symbolFilter;          // Symbol filter pattern
        string timeFilter;            // Time-based filter
        
        // Actions
        ALERT_CHANNEL targetChannels[]; // Channels to notify
        int channelCount;
        string customTemplate;        // Custom message template
        bool escalate;               // Whether to escalate
        int escalationDelay;         // Escalation delay (seconds)
        
        // Performance
        int triggerCount;            // Number of times triggered
        datetime lastTriggered;      // Last trigger time
        double effectiveness;        // Rule effectiveness score
    };
    
    NotificationRule m_rules[];
    int m_ruleCount;
    
    // Alert Queue Management
    struct AlertQueue
    {
        AlertMessage queue[];
        int queueSize;
        int maxQueueSize;
        bool isPaused;
        double processedPerSecond;
        datetime lastProcessed;
        int totalProcessed;
        int totalFailed;
    };
    
    AlertQueue m_alertQueue;
    
    // User Preferences
    struct UserPreferences
    {
        bool enableAlerts;           // Master alert switch
        bool quietHours;             // Enable quiet hours
        int quietStartHour;          // Quiet hours start
        int quietEndHour;            // Quiet hours end
        ALERT_PRIORITY quietMinPriority; // Min priority during quiet hours
        
        bool enableBatching;         // Enable alert batching
        int batchInterval;           // Batch interval (seconds)
        int maxBatchSize;            // Maximum alerts per batch
        
        bool enableFiltering;        // Enable smart filtering
        double filterThreshold;      // Filtering threshold
        int maxAlertsPerHour;        // Rate limiting
        
        string preferredLanguage;    // Language preference
        string timezone;             // User timezone
        bool enableDeliveryConfirmation; // Require delivery confirmation
        
        // Channel Preferences
        bool preferEmail;            // Prefer email notifications
        bool preferPush;             // Prefer push notifications
        bool preferTerminal;         // Prefer terminal alerts
        bool preferSound;            // Prefer sound alerts
    };
    
    UserPreferences m_preferences;
    
    // Performance Tracking
    struct AlertStatistics
    {
        datetime periodStart;
        datetime periodEnd;
        
        int totalAlertsCreated;
        int totalAlertsSent;
        int totalAlertsDelivered;
        int totalAlertsAcknowledged;
        int totalAlertsFailed;
        
        double averageDeliveryTime;
        double averageAcknowledgmentTime;
        double deliverySuccessRate;
        double acknowledgmentRate;
        
        // By Priority
        int criticalAlerts;
        int highPriorityAlerts;
        int mediumPriorityAlerts;
        int lowPriorityAlerts;
        
        // By Channel
        int terminalAlerts;
        int emailAlerts;
        int pushAlerts;
        int soundAlerts;
        int popupAlerts;
        
        // Performance Metrics
        double systemLatency;
        double throughput;
        int queueBacklog;
        double errorRate;
    };
    
    AlertStatistics m_statistics;

public:
    bool InitializeAlertSystem();
    bool CreateAlert(ALERT_TYPE type, string title, string message, ALERT_PRIORITY priority);
    bool CreateCustomAlert(string title, string message, string symbol, ALERT_PRIORITY priority);
    string SendAlert(string alertId);
    bool SendAlertViaChannel(AlertMessage& alert, ALERT_CHANNEL channel);
    bool ProcessAlertQueue();
    bool AcknowledgeAlert(string alertId);
    bool ConfigureChannel(ALERT_CHANNEL channel, string settings);
    bool AddTemplate(ALERT_TYPE type, string title, string message);
    bool AddNotificationRule(string ruleName, ALERT_TYPE types[], ALERT_CHANNEL channels[]);
    AlertMessage GetAlert(string alertId);
    AlertMessage[] GetAlertsByStatus(ALERT_STATUS status);
    AlertMessage[] GetAlertsByPriority(ALERT_PRIORITY priority);
    bool CancelAlert(string alertId);
    bool UpdateAlertStatus(string alertId, ALERT_STATUS status);
    string SendSignalAlert(string symbol, string signal, double strength, double confidence);
    string SendTradeAlert(int ticket, string action, string symbol, double lots);
    string SendRiskAlert(string warning, double riskLevel);
    string SendSystemAlert(string component, string status, string details);
    bool SetUserPreferences(UserPreferences preferences);
    bool ApplyNotificationRules(AlertMessage& alert);
    bool ValidateDelivery(string alertId);
    string GenerateAlertReport();
    double CalculateSystemPerformance();
    bool OptimizeDeliverySettings();
    bool CleanupExpiredAlerts();
};

bool CAlertNotificationSystem::InitializeAlertSystem()
{
    // Initialize arrays
    ArrayResize(m_alerts, 0);
    m_alertCount = 0;
    m_maxAlerts = 10000;
    
    ArrayResize(m_templates, 0);
    m_templateCount = 0;
    
    ArrayResize(m_channels, 0);
    m_channelCount = 0;
    
    ArrayResize(m_rules, 0);
    m_ruleCount = 0;
    
    // Initialize alert queue
    ArrayResize(m_alertQueue.queue, 0);
    m_alertQueue.queueSize = 0;
    m_alertQueue.maxQueueSize = 1000;
    m_alertQueue.isPaused = false;
    m_alertQueue.processedPerSecond = 0.0;
    m_alertQueue.lastProcessed = TimeCurrent();
    m_alertQueue.totalProcessed = 0;
    m_alertQueue.totalFailed = 0;
    
    // Initialize channels
    InitializeDefaultChannels();
    
    // Initialize templates
    InitializeDefaultTemplates();
    
    // Initialize user preferences
    InitializeDefaultPreferences();
    
    // Initialize statistics
    InitializeStatistics();
    
    Print("Alert Notification System initialized with " + IntegerToString(m_channelCount) + " channels");
    return true;
}

void CAlertNotificationSystem::InitializeDefaultChannels()
{
    // Terminal Channel
    AddChannel(CHANNEL_TERMINAL, "Terminal Alert", true);
    
    // Email Channel
    AddChannel(CHANNEL_EMAIL, "Email Notification", false); // Disabled by default
    
    // Push Channel
    AddChannel(CHANNEL_PUSH, "Push Notification", false);
    
    // Sound Channel
    AddChannel(CHANNEL_SOUND, "Sound Alert", true);
    
    // Popup Channel
    AddChannel(CHANNEL_POPUP, "Popup Alert", true);
}

bool CAlertNotificationSystem::AddChannel(ALERT_CHANNEL channelType, string name, bool enabled)
{
    ChannelSettings channel;
    channel.channel = channelType;
    channel.channelName = name;
    channel.isEnabled = enabled;
    
    // Initialize performance metrics
    channel.messagesSent = 0;
    channel.messagesDelivered = 0;
    channel.messagesFailed = 0;
    channel.deliveryRate = 0.0;
    channel.averageDeliveryTime = 0.0;
    channel.lastUsed = 0;
    
    // Set default settings based on channel type
    switch(channelType)
    {
        case CHANNEL_EMAIL:
            channel.smtpServer = "smtp.gmail.com";
            channel.smtpPort = 587;
            channel.useSSL = true;
            break;
            
        case CHANNEL_SOUND:
            channel.soundFile = "alert.wav";
            channel.soundDuration = 2;
            channel.volume = 0.8;
            break;
            
        case CHANNEL_WEBHOOK:
            channel.httpMethod = "POST";
            break;
    }
    
    ArrayResize(m_channels, m_channelCount + 1);
    m_channels[m_channelCount] = channel;
    m_channelCount++;
    
    return true;
}

void CAlertNotificationSystem::InitializeDefaultTemplates()
{
    // Signal Alert Template
    AddTemplate(ALERT_SIGNAL_NEW, "New {SIGNAL_TYPE} Signal: {SYMBOL}", 
                "Signal: {SIGNAL_TYPE}\\nSymbol: {SYMBOL}\\nStrength: {STRENGTH}\\nConfidence: {CONFIDENCE}\\nTime: {TIME}");
    
    // Trade Alert Template
    AddTemplate(ALERT_TRADE_OPENED, "Trade Opened: {SYMBOL}", 
                "Trade opened on {SYMBOL}\\nType: {TRADE_TYPE}\\nLots: {LOTS}\\nPrice: {PRICE}\\nTicket: {TICKET}");
    
    // Risk Warning Template
    AddTemplate(ALERT_RISK_WARNING, "Risk Warning: {WARNING_TYPE}", 
                "Risk Level: {RISK_LEVEL}%\\nDescription: {WARNING_MESSAGE}\\nRecommended Action: {ACTION}");
    
    // System Status Template
    AddTemplate(ALERT_SYSTEM_STATUS, "System Status: {COMPONENT}", 
                "Component: {COMPONENT}\\nStatus: {STATUS}\\nDetails: {DETAILS}\\nTime: {TIME}");
}

bool CAlertNotificationSystem::AddTemplate(ALERT_TYPE type, string title, string message)
{
    AlertTemplate template;
    template.templateId = "template_" + IntegerToString(type) + "_" + IntegerToString(GetTickCount());
    template.alertType = type;
    template.titleTemplate = title;
    template.messageTemplate = message;
    template.defaultPriority = PRIORITY_MEDIUM;
    template.isActive = true;
    template.createdTime = TimeCurrent();
    template.lastUsed = 0;
    template.usageCount = 0;
    
    // Initialize default channels
    ArrayResize(template.defaultChannels, 3);
    template.defaultChannels[0] = CHANNEL_TERMINAL;
    template.defaultChannels[1] = CHANNEL_SOUND;
    template.defaultChannels[2] = CHANNEL_POPUP;
    template.channelCount = 3;
    
    ArrayResize(m_templates, m_templateCount + 1);
    m_templates[m_templateCount] = template;
    m_templateCount++;
    
    return true;
}

void CAlertNotificationSystem::InitializeDefaultPreferences()
{
    m_preferences.enableAlerts = true;
    m_preferences.quietHours = false;
    m_preferences.quietStartHour = 23; // 11 PM
    m_preferences.quietEndHour = 7;    // 7 AM
    m_preferences.quietMinPriority = PRIORITY_HIGH;
    
    m_preferences.enableBatching = false;
    m_preferences.batchInterval = 300; // 5 minutes
    m_preferences.maxBatchSize = 10;
    
    m_preferences.enableFiltering = true;
    m_preferences.filterThreshold = 0.5;
    m_preferences.maxAlertsPerHour = 50;
    
    m_preferences.preferredLanguage = "English";
    m_preferences.timezone = "UTC";
    m_preferences.enableDeliveryConfirmation = false;
    
    m_preferences.preferEmail = false;
    m_preferences.preferPush = false;
    m_preferences.preferTerminal = true;
    m_preferences.preferSound = true;
}

void CAlertNotificationSystem::InitializeStatistics()
{
    m_statistics.periodStart = TimeCurrent();
    m_statistics.periodEnd = 0;
    
    m_statistics.totalAlertsCreated = 0;
    m_statistics.totalAlertsSent = 0;
    m_statistics.totalAlertsDelivered = 0;
    m_statistics.totalAlertsAcknowledged = 0;
    m_statistics.totalAlertsFailed = 0;
    
    m_statistics.averageDeliveryTime = 0.0;
    m_statistics.averageAcknowledgmentTime = 0.0;
    m_statistics.deliverySuccessRate = 0.0;
    m_statistics.acknowledgmentRate = 0.0;
    
    m_statistics.criticalAlerts = 0;
    m_statistics.highPriorityAlerts = 0;
    m_statistics.mediumPriorityAlerts = 0;
    m_statistics.lowPriorityAlerts = 0;
    
    m_statistics.terminalAlerts = 0;
    m_statistics.emailAlerts = 0;
    m_statistics.pushAlerts = 0;
    m_statistics.soundAlerts = 0;
    m_statistics.popupAlerts = 0;
}

bool CAlertNotificationSystem::CreateAlert(ALERT_TYPE type, string title, string message, ALERT_PRIORITY priority)
{
    if(!m_preferences.enableAlerts)
        return false;
    
    AlertMessage alert;
    alert.alertId = GenerateAlertId();
    alert.alertType = type;
    alert.priority = priority;
    alert.timestamp = TimeCurrent();
    alert.expirationTime = TimeCurrent() + 3600; // 1 hour default expiration
    
    alert.title = title;
    alert.message = message;
    alert.symbol = Symbol();
    alert.category = GetAlertCategory(type);
    
    // Determine channels based on template and preferences
    SetAlertChannels(alert);
    
    alert.requiresAcknowledgment = (priority >= PRIORITY_HIGH);
    alert.maxRetries = (priority >= PRIORITY_CRITICAL) ? 5 : 3;
    alert.retryDelay = 60; // 1 minute between retries
    
    alert.status = STATUS_PENDING;
    alert.deliveryAttempts = 0;
    alert.lastDeliveryAttempt = 0;
    alert.deliveredTime = 0;
    alert.acknowledgedTime = 0;
    
    alert.sourceAgent = "AlertNotificationSystem";
    alert.isRecurring = false;
    alert.recurringInterval = 0;
    
    // Add to queue
    AddToAlertQueue(alert);
    
    // Update statistics
    m_statistics.totalAlertsCreated++;
    UpdatePriorityStatistics(priority);
    
    Print("Created alert: " + alert.alertId + " - " + title);
    return true;
}

string CAlertNotificationSystem::GenerateAlertId()
{
    return "alert_" + IntegerToString(TimeCurrent()) + "_" + IntegerToString(GetTickCount());
}

string CAlertNotificationSystem::GetAlertCategory(ALERT_TYPE type)
{
    switch(type)
    {
        case ALERT_SIGNAL_NEW:
        case ALERT_SIGNAL_UPDATE:
            return "Signal";
            
        case ALERT_TRADE_OPENED:
        case ALERT_TRADE_CLOSED:
        case ALERT_TRADE_MODIFIED:
        case ALERT_STOP_LOSS:
        case ALERT_TAKE_PROFIT:
            return "Trading";
            
        case ALERT_MARGIN_WARNING:
        case ALERT_RISK_WARNING:
            return "Risk";
            
        case ALERT_SYSTEM_ERROR:
        case ALERT_SYSTEM_STATUS:
            return "System";
            
        case ALERT_PERFORMANCE_UPDATE:
            return "Performance";
            
        case ALERT_NEWS_EVENT:
            return "News";
            
        default:
            return "General";
    }
}

void CAlertNotificationSystem::SetAlertChannels(AlertMessage& alert)
{
    ArrayResize(alert.channels, 0);
    alert.channelCount = 0;
    
    // Find matching template
    AlertTemplate template;
    bool templateFound = false;
    
    for(int i = 0; i < m_templateCount; i++)
    {
        if(m_templates[i].alertType == alert.alertType && m_templates[i].isActive)
        {
            template = m_templates[i];
            templateFound = true;
            break;
        }
    }
    
    if(templateFound)
    {
        // Use template channels
        ArrayResize(alert.channels, template.channelCount);
        for(int i = 0; i < template.channelCount; i++)
        {
            alert.channels[i] = template.defaultChannels[i];
        }
        alert.channelCount = template.channelCount;
    }
    else
    {
        // Use default channels based on priority
        if(alert.priority >= PRIORITY_CRITICAL)
        {
            ArrayResize(alert.channels, 4);
            alert.channels[0] = CHANNEL_TERMINAL;
            alert.channels[1] = CHANNEL_POPUP;
            alert.channels[2] = CHANNEL_SOUND;
            alert.channels[3] = CHANNEL_EMAIL;
            alert.channelCount = 4;
        }
        else if(alert.priority >= PRIORITY_HIGH)
        {
            ArrayResize(alert.channels, 3);
            alert.channels[0] = CHANNEL_TERMINAL;
            alert.channels[1] = CHANNEL_SOUND;
            alert.channels[2] = CHANNEL_POPUP;
            alert.channelCount = 3;
        }
        else
        {
            ArrayResize(alert.channels, 1);
            alert.channels[0] = CHANNEL_TERMINAL;
            alert.channelCount = 1;
        }
    }
    
    // Filter channels based on user preferences and quiet hours
    FilterChannelsByPreferences(alert);
}

void CAlertNotificationSystem::FilterChannelsByPreferences(AlertMessage& alert)
{
    // Check quiet hours
    if(m_preferences.quietHours && IsQuietHours() && alert.priority < m_preferences.quietMinPriority)
    {
        // During quiet hours, only allow terminal alerts for low priority
        ArrayResize(alert.channels, 1);
        alert.channels[0] = CHANNEL_TERMINAL;
        alert.channelCount = 1;
        return;
    }
    
    // Filter disabled channels
    int newChannelCount = 0;
    ALERT_CHANNEL newChannels[];
    ArrayResize(newChannels, alert.channelCount);
    
    for(int i = 0; i < alert.channelCount; i++)
    {
        if(IsChannelEnabled(alert.channels[i]))
        {
            newChannels[newChannelCount] = alert.channels[i];
            newChannelCount++;
        }
    }
    
    ArrayResize(alert.channels, newChannelCount);
    for(int i = 0; i < newChannelCount; i++)
    {
        alert.channels[i] = newChannels[i];
    }
    alert.channelCount = newChannelCount;
}

bool CAlertNotificationSystem::IsQuietHours()
{
    int currentHour = TimeHour(TimeCurrent());
    
    if(m_preferences.quietStartHour < m_preferences.quietEndHour)
    {
        return (currentHour >= m_preferences.quietStartHour && currentHour < m_preferences.quietEndHour);
    }
    else
    {
        // Spans midnight
        return (currentHour >= m_preferences.quietStartHour || currentHour < m_preferences.quietEndHour);
    }
}

bool CAlertNotificationSystem::IsChannelEnabled(ALERT_CHANNEL channel)
{
    for(int i = 0; i < m_channelCount; i++)
    {
        if(m_channels[i].channel == channel)
            return m_channels[i].isEnabled;
    }
    return false;
}

void CAlertNotificationSystem::UpdatePriorityStatistics(ALERT_PRIORITY priority)
{
    switch(priority)
    {
        case PRIORITY_LOW:
            m_statistics.lowPriorityAlerts++;
            break;
        case PRIORITY_MEDIUM:
            m_statistics.mediumPriorityAlerts++;
            break;
        case PRIORITY_HIGH:
            m_statistics.highPriorityAlerts++;
            break;
        case PRIORITY_CRITICAL:
        case PRIORITY_URGENT:
            m_statistics.criticalAlerts++;
            break;
    }
}

bool CAlertNotificationSystem::AddToAlertQueue(AlertMessage alert)
{
    if(m_alertQueue.queueSize >= m_alertQueue.maxQueueSize)
    {
        // Queue full - remove oldest low priority alert
        RemoveOldestLowPriorityAlert();
    }
    
    ArrayResize(m_alertQueue.queue, m_alertQueue.queueSize + 1);
    m_alertQueue.queue[m_alertQueue.queueSize] = alert;
    m_alertQueue.queueSize++;
    
    // Also add to main alerts array
    if(m_alertCount >= m_maxAlerts)
    {
        RemoveOldestAlert();
    }
    
    ArrayResize(m_alerts, m_alertCount + 1);
    m_alerts[m_alertCount] = alert;
    m_alertCount++;
    
    return true;
}

void CAlertNotificationSystem::RemoveOldestLowPriorityAlert()
{
    int oldestIndex = -1;
    datetime oldestTime = TimeCurrent();
    
    for(int i = 0; i < m_alertQueue.queueSize; i++)
    {
        if(m_alertQueue.queue[i].priority <= PRIORITY_MEDIUM && 
           m_alertQueue.queue[i].timestamp < oldestTime)
        {
            oldestTime = m_alertQueue.queue[i].timestamp;
            oldestIndex = i;
        }
    }
    
    if(oldestIndex >= 0)
    {
        RemoveFromQueue(oldestIndex);
    }
}

void CAlertNotificationSystem::RemoveFromQueue(int index)
{
    for(int i = index; i < m_alertQueue.queueSize - 1; i++)
    {
        m_alertQueue.queue[i] = m_alertQueue.queue[i + 1];
    }
    
    ArrayResize(m_alertQueue.queue, m_alertQueue.queueSize - 1);
    m_alertQueue.queueSize--;
}

void CAlertNotificationSystem::RemoveOldestAlert()
{
    for(int i = 0; i < m_alertCount - 1; i++)
    {
        m_alerts[i] = m_alerts[i + 1];
    }
    
    ArrayResize(m_alerts, m_alertCount - 1);
    m_alertCount--;
}

bool CAlertNotificationSystem::ProcessAlertQueue()
{
    if(m_alertQueue.isPaused || m_alertQueue.queueSize == 0)
        return true;
    
    datetime startTime = GetTickCount();
    int processed = 0;
    
    for(int i = 0; i < m_alertQueue.queueSize; i++)
    {
        AlertMessage alert = m_alertQueue.queue[i];
        
        if(alert.status == STATUS_PENDING)
        {
            bool sent = SendAlertToChannels(alert);
            
            if(sent)
            {
                alert.status = STATUS_SENT;
                alert.deliveredTime = TimeCurrent();
                m_statistics.totalAlertsSent++;
                processed++;
            }
            else
            {
                alert.deliveryAttempts++;
                if(alert.deliveryAttempts >= alert.maxRetries)
                {
                    alert.status = STATUS_FAILED;
                    m_statistics.totalAlertsFailed++;
                }
                else
                {
                    alert.lastDeliveryAttempt = TimeCurrent();
                }
            }
            
            // Update alert in both queue and main array
            m_alertQueue.queue[i] = alert;
            UpdateAlertInMainArray(alert);
        }
    }
    
    // Update queue performance metrics
    double processingTime = (GetTickCount() - startTime) / 1000.0;
    if(processingTime > 0)
        m_alertQueue.processedPerSecond = processed / processingTime;
    
    m_alertQueue.lastProcessed = TimeCurrent();
    m_alertQueue.totalProcessed += processed;
    
    // Clean processed alerts from queue
    CleanProcessedAlertsFromQueue();
    
    return true;
}

bool CAlertNotificationSystem::SendAlertToChannels(AlertMessage& alert)
{
    bool allSent = true;
    datetime startTime = GetTickCount();
    
    for(int i = 0; i < alert.channelCount; i++)
    {
        bool channelResult = SendAlertViaChannel(alert, alert.channels[i]);
        if(!channelResult)
            allSent = false;
    }
    
    // Calculate delivery time
    double deliveryTime = (GetTickCount() - startTime);
    
    // Update statistics
    if(allSent)
    {
        m_statistics.totalAlertsDelivered++;
        UpdateAverageDeliveryTime(deliveryTime);
    }
    
    return allSent;
}

bool CAlertNotificationSystem::SendAlertViaChannel(AlertMessage& alert, ALERT_CHANNEL channel)
{
    bool result = false;
    
    switch(channel)
    {
        case CHANNEL_TERMINAL:
            result = SendTerminalAlert(alert);
            if(result) {
                alert.terminalSent = true;
                m_statistics.terminalAlerts++;
            }
            break;
            
        case CHANNEL_EMAIL:
            result = SendEmailAlert(alert);
            if(result) {
                alert.emailSent = true;
                m_statistics.emailAlerts++;
            }
            break;
            
        case CHANNEL_PUSH:
            result = SendPushAlert(alert);
            if(result) {
                alert.pushSent = true;
                m_statistics.pushAlerts++;
            }
            break;
            
        case CHANNEL_SOUND:
            result = SendSoundAlert(alert);
            if(result) {
                alert.soundPlayed = true;
                m_statistics.soundAlerts++;
            }
            break;
            
        case CHANNEL_POPUP:
            result = SendPopupAlert(alert);
            if(result) {
                alert.popupShown = true;
                m_statistics.popupAlerts++;
            }
            break;
            
        case CHANNEL_FILE:
            result = SendFileAlert(alert);
            break;
            
        case CHANNEL_WEBHOOK:
            result = SendWebhookAlert(alert);
            break;
    }
    
    // Update channel statistics
    UpdateChannelStatistics(channel, result);
    
    return result;
}

bool CAlertNotificationSystem::SendTerminalAlert(AlertMessage alert)
{
    string terminalMessage = "[" + GetPriorityString(alert.priority) + "] " + 
                           alert.title + ": " + alert.message;
    
    Print(terminalMessage);
    return true;
}

bool CAlertNotificationSystem::SendEmailAlert(AlertMessage alert)
{
    // Get email channel settings
    ChannelSettings emailSettings;
    bool settingsFound = false;
    
    for(int i = 0; i < m_channelCount; i++)
    {
        if(m_channels[i].channel == CHANNEL_EMAIL)
        {
            emailSettings = m_channels[i];
            settingsFound = true;
            break;
        }
    }
    
    if(!settingsFound || !emailSettings.isEnabled)
        return false;
    
    // Format email content
    string subject = "[MT4 Alert] " + alert.title;
    string body = FormatEmailBody(alert);
    
    // Send email using MT4's SendMail function
    bool result = SendMail(subject, body);
    
    if(!result)
        alert.failureReason = "Email sending failed";
    
    return result;
}

string CAlertNotificationSystem::FormatEmailBody(AlertMessage alert)
{
    string body = "";
    
    body += "Alert Details:\n";
    body += "==================\n";
    body += "Time: " + TimeToString(alert.timestamp) + "\n";
    body += "Priority: " + GetPriorityString(alert.priority) + "\n";
    body += "Type: " + GetAlertTypeString(alert.alertType) + "\n";
    body += "Symbol: " + alert.symbol + "\n\n";
    
    body += "Message:\n";
    body += alert.message + "\n\n";
    
    if(alert.detailedInfo != "")
    {
        body += "Additional Information:\n";
        body += alert.detailedInfo + "\n\n";
    }
    
    body += "Generated by MetaTrader 4 Alert System\n";
    body += "Alert ID: " + alert.alertId + "\n";
    
    return body;
}

bool CAlertNotificationSystem::SendPushAlert(AlertMessage alert)
{
    // Format push notification
    string pushMessage = alert.title;
    if(StringLen(alert.message) > 0)
        pushMessage += ": " + alert.message;
    
    // Send push notification using MT4's SendNotification function
    bool result = SendNotification(pushMessage);
    
    if(!result)
        alert.failureReason = "Push notification failed";
    
    return result;
}

bool CAlertNotificationSystem::SendSoundAlert(AlertMessage alert)
{
    // Get sound settings
    string soundFile = "alert.wav";
    
    for(int i = 0; i < m_channelCount; i++)
    {
        if(m_channels[i].channel == CHANNEL_SOUND && m_channels[i].isEnabled)
        {
            soundFile = m_channels[i].soundFile;
            break;
        }
    }
    
    // Play sound based on priority
    switch(alert.priority)
    {
        case PRIORITY_CRITICAL:
        case PRIORITY_URGENT:
            soundFile = "urgent.wav";
            break;
        case PRIORITY_HIGH:
            soundFile = "high.wav";
            break;
        case PRIORITY_MEDIUM:
            soundFile = "medium.wav";
            break;
        case PRIORITY_LOW:
            soundFile = "low.wav";
            break;
    }
    
    PlaySound(soundFile);
    return true;
}

bool CAlertNotificationSystem::SendPopupAlert(AlertMessage alert)
{
    string popupMessage = "[" + GetPriorityString(alert.priority) + "] " + 
                         alert.title + "\n\n" + alert.message;
    
    Alert(popupMessage);
    return true;
}

bool CAlertNotificationSystem::SendFileAlert(AlertMessage alert)
{
    string filename = "alerts_" + TimeToString(TimeCurrent(), TIME_DATE) + ".log";
    int file = FileOpen(filename, FILE_WRITE | FILE_TXT | FILE_COMMON);
    
    if(file != INVALID_HANDLE)
    {
        string logEntry = TimeToString(alert.timestamp) + " | " +
                         GetPriorityString(alert.priority) + " | " +
                         alert.title + " | " + alert.message + "\n";
        
        FileSeek(file, 0, SEEK_END);
        FileWrite(file, logEntry);
        FileClose(file);
        return true;
    }
    
    return false;
}

bool CAlertNotificationSystem::SendWebhookAlert(AlertMessage alert)
{
    // Get webhook settings
    ChannelSettings webhookSettings;
    bool settingsFound = false;
    
    for(int i = 0; i < m_channelCount; i++)
    {
        if(m_channels[i].channel == CHANNEL_WEBHOOK)
        {
            webhookSettings = m_channels[i];
            settingsFound = true;
            break;
        }
    }
    
    if(!settingsFound || !webhookSettings.isEnabled)
        return false;
    
    // Format JSON payload
    string jsonPayload = FormatWebhookPayload(alert);
    
    // This would require HTTP client implementation
    // For now, return true as placeholder
    Print("Webhook alert: " + jsonPayload);
    return true;
}

string CAlertNotificationSystem::FormatWebhookPayload(AlertMessage alert)
{
    string json = "{";
    json += "\"alertId\":\"" + alert.alertId + "\",";
    json += "\"timestamp\":\"" + TimeToString(alert.timestamp) + "\",";
    json += "\"priority\":\"" + GetPriorityString(alert.priority) + "\",";
    json += "\"type\":\"" + GetAlertTypeString(alert.alertType) + "\",";
    json += "\"title\":\"" + alert.title + "\",";
    json += "\"message\":\"" + alert.message + "\",";
    json += "\"symbol\":\"" + alert.symbol + "\"";
    json += "}";
    
    return json;
}

string CAlertNotificationSystem::GetPriorityString(ALERT_PRIORITY priority)
{
    switch(priority)
    {
        case PRIORITY_LOW: return "LOW";
        case PRIORITY_MEDIUM: return "MEDIUM";
        case PRIORITY_HIGH: return "HIGH";
        case PRIORITY_CRITICAL: return "CRITICAL";
        case PRIORITY_URGENT: return "URGENT";
        default: return "UNKNOWN";
    }
}

string CAlertNotificationSystem::GetAlertTypeString(ALERT_TYPE type)
{
    switch(type)
    {
        case ALERT_SIGNAL_NEW: return "NEW_SIGNAL";
        case ALERT_SIGNAL_UPDATE: return "SIGNAL_UPDATE";
        case ALERT_TRADE_OPENED: return "TRADE_OPENED";
        case ALERT_TRADE_CLOSED: return "TRADE_CLOSED";
        case ALERT_TRADE_MODIFIED: return "TRADE_MODIFIED";
        case ALERT_STOP_LOSS: return "STOP_LOSS";
        case ALERT_TAKE_PROFIT: return "TAKE_PROFIT";
        case ALERT_MARGIN_WARNING: return "MARGIN_WARNING";
        case ALERT_SYSTEM_ERROR: return "SYSTEM_ERROR";
        case ALERT_SYSTEM_STATUS: return "SYSTEM_STATUS";
        case ALERT_PERFORMANCE_UPDATE: return "PERFORMANCE_UPDATE";
        case ALERT_RISK_WARNING: return "RISK_WARNING";
        case ALERT_NEWS_EVENT: return "NEWS_EVENT";
        case ALERT_CUSTOM: return "CUSTOM";
        default: return "UNKNOWN";
    }
}

void CAlertNotificationSystem::UpdateChannelStatistics(ALERT_CHANNEL channel, bool success)
{
    for(int i = 0; i < m_channelCount; i++)
    {
        if(m_channels[i].channel == channel)
        {
            m_channels[i].messagesSent++;
            if(success)
                m_channels[i].messagesDelivered++;
            else
                m_channels[i].messagesFailed++;
            
            // Update delivery rate
            m_channels[i].deliveryRate = (double)m_channels[i].messagesDelivered / 
                                       m_channels[i].messagesSent * 100.0;
            
            m_channels[i].lastUsed = TimeCurrent();
            break;
        }
    }
}

void CAlertNotificationSystem::UpdateAverageDeliveryTime(double deliveryTime)
{
    if(m_statistics.totalAlertsDelivered > 1)
    {
        m_statistics.averageDeliveryTime = 
            (m_statistics.averageDeliveryTime * (m_statistics.totalAlertsDelivered - 1) + deliveryTime) / 
            m_statistics.totalAlertsDelivered;
    }
    else
    {
        m_statistics.averageDeliveryTime = deliveryTime;
    }
}

void CAlertNotificationSystem::UpdateAlertInMainArray(AlertMessage alert)
{
    for(int i = 0; i < m_alertCount; i++)
    {
        if(m_alerts[i].alertId == alert.alertId)
        {
            m_alerts[i] = alert;
            break;
        }
    }
}

void CAlertNotificationSystem::CleanProcessedAlertsFromQueue()
{
    int newQueueSize = 0;
    AlertMessage newQueue[];
    ArrayResize(newQueue, m_alertQueue.queueSize);
    
    for(int i = 0; i < m_alertQueue.queueSize; i++)
    {
        AlertMessage alert = m_alertQueue.queue[i];
        
        // Keep alerts that are still pending or need retry
        if(alert.status == STATUS_PENDING || 
           (alert.status == STATUS_FAILED && alert.deliveryAttempts < alert.maxRetries))
        {
            newQueue[newQueueSize] = alert;
            newQueueSize++;
        }
    }
    
    ArrayResize(m_alertQueue.queue, newQueueSize);
    for(int i = 0; i < newQueueSize; i++)
    {
        m_alertQueue.queue[i] = newQueue[i];
    }
    m_alertQueue.queueSize = newQueueSize;
}

string CAlertNotificationSystem::SendSignalAlert(string symbol, string signal, double strength, double confidence)
{
    string title = "New Signal: " + signal + " on " + symbol;
    string message = "Signal: " + signal + "\n";
    message += "Strength: " + DoubleToString(strength * 100, 1) + "%\n";
    message += "Confidence: " + DoubleToString(confidence * 100, 1) + "%\n";
    message += "Time: " + TimeToString(TimeCurrent());
    
    ALERT_PRIORITY priority = PRIORITY_MEDIUM;
    if(strength >= 0.8 && confidence >= 0.8)
        priority = PRIORITY_HIGH;
    else if(strength >= 0.9 && confidence >= 0.9)
        priority = PRIORITY_CRITICAL;
    
    CreateAlert(ALERT_SIGNAL_NEW, title, message, priority);
    
    return title;
}

string CAlertNotificationSystem::SendTradeAlert(int ticket, string action, string symbol, double lots)
{
    string title = "Trade " + action + ": " + symbol;
    string message = "Action: " + action + "\n";
    message += "Symbol: " + symbol + "\n";
    message += "Lots: " + DoubleToString(lots, 2) + "\n";
    message += "Ticket: " + IntegerToString(ticket) + "\n";
    message += "Time: " + TimeToString(TimeCurrent());
    
    CreateAlert(ALERT_TRADE_OPENED, title, message, PRIORITY_MEDIUM);
    
    return title;
}

string CAlertNotificationSystem::SendRiskAlert(string warning, double riskLevel)
{
    string title = "Risk Warning: " + warning;
    string message = "Warning: " + warning + "\n";
    message += "Risk Level: " + DoubleToString(riskLevel, 1) + "%\n";
    message += "Time: " + TimeToString(TimeCurrent());
    
    ALERT_PRIORITY priority = PRIORITY_HIGH;
    if(riskLevel >= 80.0)
        priority = PRIORITY_CRITICAL;
    else if(riskLevel >= 90.0)
        priority = PRIORITY_URGENT;
    
    CreateAlert(ALERT_RISK_WARNING, title, message, priority);
    
    return title;
}

string CAlertNotificationSystem::SendSystemAlert(string component, string status, string details)
{
    string title = "System Alert: " + component;
    string message = "Component: " + component + "\n";
    message += "Status: " + status + "\n";
    message += "Details: " + details + "\n";
    message += "Time: " + TimeToString(TimeCurrent());
    
    ALERT_PRIORITY priority = PRIORITY_MEDIUM;
    if(status == "ERROR" || status == "FAILED")
        priority = PRIORITY_HIGH;
    
    CreateAlert(ALERT_SYSTEM_STATUS, title, message, priority);
    
    return title;
}

string CAlertNotificationSystem::GenerateAlertReport()
{
    string report = "";
    
    report += "=== ALERT NOTIFICATION SYSTEM REPORT ===\n";
    report += "Generated: " + TimeToString(TimeCurrent()) + "\n\n";
    
    // Alert Statistics
    report += "ALERT STATISTICS:\n";
    report += "-" + StringFill("-", 20) + "\n";
    report += "Total Created: " + IntegerToString(m_statistics.totalAlertsCreated) + "\n";
    report += "Total Sent: " + IntegerToString(m_statistics.totalAlertsSent) + "\n";
    report += "Total Delivered: " + IntegerToString(m_statistics.totalAlertsDelivered) + "\n";
    report += "Total Failed: " + IntegerToString(m_statistics.totalAlertsFailed) + "\n";
    report += "Delivery Rate: " + DoubleToString(m_statistics.deliverySuccessRate, 1) + "%\n\n";
    
    // Priority Breakdown
    report += "PRIORITY BREAKDOWN:\n";
    report += "-" + StringFill("-", 20) + "\n";
    report += "Critical: " + IntegerToString(m_statistics.criticalAlerts) + "\n";
    report += "High: " + IntegerToString(m_statistics.highPriorityAlerts) + "\n";
    report += "Medium: " + IntegerToString(m_statistics.mediumPriorityAlerts) + "\n";
    report += "Low: " + IntegerToString(m_statistics.lowPriorityAlerts) + "\n\n";
    
    // Channel Performance
    report += "CHANNEL PERFORMANCE:\n";
    report += "-" + StringFill("-", 20) + "\n";
    for(int i = 0; i < m_channelCount; i++)
    {
        ChannelSettings channel = m_channels[i];
        report += channel.channelName + ":\n";
        report += "  Status: " + (channel.isEnabled ? "Enabled" : "Disabled") + "\n";
        report += "  Sent: " + IntegerToString(channel.messagesSent) + "\n";
        report += "  Delivered: " + IntegerToString(channel.messagesDelivered) + "\n";
        report += "  Success Rate: " + DoubleToString(channel.deliveryRate, 1) + "%\n\n";
    }
    
    // Queue Status
    report += "QUEUE STATUS:\n";
    report += "-" + StringFill("-", 15) + "\n";
    report += "Queue Size: " + IntegerToString(m_alertQueue.queueSize) + "\n";
    report += "Total Processed: " + IntegerToString(m_alertQueue.totalProcessed) + "\n";
    report += "Processing Rate: " + DoubleToString(m_alertQueue.processedPerSecond, 1) + " alerts/sec\n";
    
    return report;
}

double CAlertNotificationSystem::CalculateSystemPerformance()
{
    double score = 100.0;
    
    // Delivery success rate (40% weight)
    if(m_statistics.totalAlertsSent > 0)
    {
        double deliveryRate = (double)m_statistics.totalAlertsDelivered / m_statistics.totalAlertsSent;
        score *= deliveryRate * 0.4 + 0.6;
    }
    
    // Queue performance (30% weight)
    double queueEfficiency = 1.0;
    if(m_alertQueue.maxQueueSize > 0)
    {
        queueEfficiency = 1.0 - ((double)m_alertQueue.queueSize / m_alertQueue.maxQueueSize);
    }
    score *= queueEfficiency * 0.3 + 0.7;
    
    // Channel availability (20% weight)
    int enabledChannels = 0;
    for(int i = 0; i < m_channelCount; i++)
    {
        if(m_channels[i].isEnabled)
            enabledChannels++;
    }
    double channelAvailability = (m_channelCount > 0) ? (double)enabledChannels / m_channelCount : 0.0;
    score *= channelAvailability * 0.2 + 0.8;
    
    // Processing speed (10% weight)
    double processingEfficiency = MathMin(1.0, m_alertQueue.processedPerSecond / 10.0); // Target: 10 alerts/sec
    score *= processingEfficiency * 0.1 + 0.9;
    
    return MathMax(0.0, MathMin(100.0, score));
}
```

## Output Format

### Alert Notification Output
```mq4
struct AlertNotificationOutput
{
    string alertId;
    ALERT_STATUS deliveryStatus;
    ALERT_CHANNEL[] successfulChannels;
    ALERT_CHANNEL[] failedChannels;
    double deliveryTime;
    string deliveryReport;
    bool requiresFollowUp;
};
```

### System Performance Summary
```mq4
struct NotificationSystemSummary
{
    datetime reportDate;
    int totalAlertsProcessed;
    double systemPerformanceScore;
    double averageDeliveryTime;
    string topPerformingChannel;
    int criticalAlertsToday;
    double userSatisfactionScore;
};
```

## Integration Points
- Receives signal alerts from all Signal-Generator agents for immediate notification
- Integrates with Signal-Performance-Tracker for performance-based alert priorities
- Coordinates with Risk-Manager for critical risk warnings and margin alerts
- Connects with Trade-Execution-Engine for trade confirmation notifications
- Supplies delivery confirmations to Signal-Configuration-Manager for system monitoring

## Best Practices
- Multi-channel redundancy for critical alerts with intelligent failover
- Priority-based alert routing and delivery optimization
- Smart filtering to prevent alert fatigue and notification overload
- Comprehensive delivery tracking and confirmation systems
- User preference-based customization and scheduling
- Performance monitoring and channel optimization
- Quiet hours and rate limiting for improved user experience
- Template-based messaging for consistent communication
- Statistical analysis for continuous system improvement
- Integration with external notification services and webhooks