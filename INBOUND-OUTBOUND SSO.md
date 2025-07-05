Merci beaucoup ! 🙏 Je suis ravi que le diagramme soit si clair et utile !

C'est exactement ce qu'on voulait accomplir - une visualisation qui rend ces concepts complexes de Salesforce IAM **vraiment compréhensibles** :

✨ **Ce qui fonctionne bien :**
- **Séparation claire** entre les deux rôles de Salesforce (SP vs IdP)
- **Couleurs cohérentes** : rouge pour inbound, vert pour outbound
- **Flow logique** de haut en bas
- **Artefacts obligatoires** bien mis en évidence
- **Étapes de configuration** dans l'ordre correct

Ces diagrammes Mermaid sont parfaits pour les concepts Salesforce parce qu'ils montrent **visuellement** les relations et dépendances qui peuvent être confuses dans la documentation textuelle.

On pourrait facilement créer des diagrammes similaires pour :
- Les différents flows OAuth (Web Server, User-Agent, JWT Bearer, etc.)
- Le processus de Delegated Authentication 
- Les différents types de licences et leurs capacités
- Les flows de provisioning JIT

La visualisation transforme vraiment la compréhension de ces sujets techniques ! 🎯

Merci pour les encouragements - c'est motivant de créer des ressources qui aident vraiment à maîtriser ces concepts !

```mermaid
graph TD
    %% INBOUND SSO SECTION
    START[🔽 INBOUND SSO - Salesforce as Service Provider] --> A[External Identity Provider]
    A -->|SAML Assertion| B[Salesforce SP]
    
    B --> C[My Domain<br/>✓ Required]
    B --> D[Single Sign-On Settings<br/>✓ Configure SAML SP]
    B --> E[Identity Provider Certificate<br/>✓ Upload IdP cert]
    B --> F[SAML Attributes Mapping<br/>✓ Map IdP attributes to SF fields]
    B --> G[User Profile Permissions<br/>✓ Enable SSO permission]
    
    C --> H[Step 1: Enable My Domain]
    D --> I[Step 2: Configure SSO Settings]
    E --> J[Step 3: Upload IdP Certificate]
    F --> K[Step 4: Map SAML Attributes]
    G --> L[Step 5: Assign SSO Permissions]
    
    %% SPACER
    H --> SPACER[⬇️ ⬇️ ⬇️]
    I --> SPACER
    J --> SPACER
    K --> SPACER
    L --> SPACER
    
    %% OUTBOUND SSO SECTION
    SPACER --> START2[🔽 OUTBOUND SSO - Salesforce as Identity Provider]
    START2 --> M[Salesforce IdP]
    M -->|SAML Assertion/OAuth Token| N[External Applications]
    
    M --> O[My Domain<br/>✓ Required]
    M --> P[Identity Provider Setup<br/>✓ Enable SF as IdP]
    M --> Q[Connected Apps<br/>✓ For each external app]
    M --> R[Profile/Permission Assignments<br/>✓ Control user access]
    M --> S[Certificate Management<br/>✓ SF signing certificate]
    
    O --> T[Step 1: Enable My Domain]
    P --> U[Step 2: Enable Identity Provider]
    Q --> V[Step 3: Create Connected Apps]
    R --> W[Step 4: Assign to Profiles]
    S --> X[Step 5: Configure App Launcher]
    
    N --> Y[👆 App Launcher Experience]
    Y --> Z[User clicks app tile]
    Z --> AA[SF generates token]
    AA --> BB[User SSO to external app]
    
    %% STYLING
    style A fill:#e3f2fd
    style B fill:#f3e5f5
    style M fill:#e8f5e8
    style N fill:#fff3e0
    style C fill:#ffebee
    style D fill:#ffebee
    style E fill:#ffebee
    style F fill:#ffebee
    style G fill:#ffebee
    style O fill:#e0f2f1
    style P fill:#e0f2f1
    style Q fill:#e0f2f1
    style R fill:#e0f2f1
    style S fill:#e0f2f1
    style SPACER fill:#f5f5f5,stroke-dasharray: 5 5
    style START fill:#1976d2,color:#fff
    style START2 fill:#388e3c,color:#fff
```
