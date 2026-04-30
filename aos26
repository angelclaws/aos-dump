import React, { useState, useEffect } from 'react';
import { Shield, Swords, Book, Flame, Zap, Crown, Star } from 'lucide-react';

const AOSWarscrollsApp = () => {
  const [factions, setFactions] = useState([]);
  const [selectedFaction, setSelectedFaction] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [factionData, setFactionData] = useState(null);

  // Fetch repository structure
  useEffect(() => {
    fetchRepositoryData();
  }, []);

  const fetchRepositoryData = async () => {
    try {
      setLoading(true);
      
      // Fetch the contents of the repository
      const response = await fetch(
        'https://api.github.com/repos/BSData/age-of-sigmar-4th/contents'
      );
      
      if (!response.ok) throw new Error('Failed to fetch repository data');
      
      const contents = await response.json();
      
      // Filter for .cat files (catalogue files containing faction data)
      const catFiles = contents.filter(file => 
        file.name.endsWith('.cat') && file.type === 'file'
      );
      
      // Extract faction names from filenames
      const factionList = catFiles.map(file => ({
        name: file.name.replace('.cat', '').replace(/_/g, ' ').replace(/-/g, ' '),
        fileName: file.name,
        downloadUrl: file.download_url
      }));
      
      setFactions(factionList);
      setLoading(false);
    } catch (err) {
      setError('Failed to load faction data: ' + err.message);
      setLoading(false);
    }
  };

  const fetchFactionData = async (faction) => {
    try {
      setLoading(true);
      setSelectedFaction(faction);
      
      // Fetch the XML file
      const response = await fetch(faction.downloadUrl);
      if (!response.ok) throw new Error('Failed to fetch faction file');
      
      const xmlText = await response.text();
      
      // Parse XML
      const parser = new DOMParser();
      const xmlDoc = parser.parseFromString(xmlText, 'text/xml');
      
      // Extract relevant data from XML
      const parsedData = parseXMLData(xmlDoc, faction.name);
      setFactionData(parsedData);
      setLoading(false);
    } catch (err) {
      setError('Failed to load faction details: ' + err.message);
      setLoading(false);
    }
  };

  const parseXMLData = (xmlDoc, factionName) => {
    const data = {
      name: factionName,
      battleTraits: [],
      formations: [],
      artefacts: [],
      spells: [],
      prayers: [],
      warscrolls: []
    };

    // Parse shared profiles (abilities, rules, etc.)
    const sharedProfiles = xmlDoc.querySelectorAll('sharedProfile');
    sharedProfiles.forEach(profile => {
      const typeName = profile.getAttribute('typeName');
      const name = profile.getAttribute('name');
      
      const characteristics = {};
      profile.querySelectorAll('characteristic').forEach(char => {
        characteristics[char.getAttribute('name')] = char.textContent;
      });

      const item = { name, ...characteristics };

      if (typeName?.includes('Ability') || typeName?.includes('Battle Trait')) {
        data.battleTraits.push(item);
      } else if (typeName?.includes('Artefact')) {
        data.artefacts.push(item);
      } else if (typeName?.includes('Spell')) {
        data.spells.push(item);
      } else if (typeName?.includes('Prayer')) {
        data.prayers.push(item);
      } else if (typeName?.includes('Unit') || typeName?.includes('Warscroll')) {
        data.warscrolls.push(item);
      }
    });

    // Parse selection entries for units/warscrolls
    const selectionEntries = xmlDoc.querySelectorAll('selectionEntry[type="unit"], selectionEntry[type="model"]');
    selectionEntries.forEach(entry => {
      const name = entry.getAttribute('name');
      const profiles = [];
      const abilities = [];
      
      // Get unit profiles
      entry.querySelectorAll('profile').forEach(profile => {
        const typeName = profile.getAttribute('typeName');
        const characteristics = {};
        
        profile.querySelectorAll('characteristic').forEach(char => {
          characteristics[char.getAttribute('name')] = char.textContent;
        });
        
        profiles.push({
          type: typeName,
          name: profile.getAttribute('name'),
          ...characteristics
        });
      });

      // Get abilities
      entry.querySelectorAll('infoLink[type="profile"]').forEach(link => {
        abilities.push(link.getAttribute('name'));
      });

      if (profiles.length > 0) {
        data.warscrolls.push({
          name,
          profiles,
          abilities
        });
      }
    });

    return data;
  };

  const StatBlock = ({ stats }) => (
    <div className="stat-block">
      {Object.entries(stats).map(([key, value]) => (
        <div key={key} className="stat-item">
          <div className="stat-label">{key}</div>
          <div className="stat-value">{value}</div>
        </div>
      ))}
    </div>
  );

  const AbilityCard = ({ ability, icon: Icon }) => (
    <div className="ability-card">
      <div className="ability-header">
        {Icon && <Icon className="ability-icon" size={20} />}
        <h4>{ability.name}</h4>
      </div>
      <div className="ability-content">
        {Object.entries(ability).map(([key, value]) => {
          if (key === 'name') return null;
          return (
            <div key={key} className="ability-section">
              <strong>{key}:</strong> {value}
            </div>
          );
        })}
      </div>
    </div>
  );

  const WarscrollCard = ({ warscroll }) => (
    <div className="warscroll-card">
      <div className="warscroll-header">
        <Shield className="warscroll-icon" size={24} />
        <h3>{warscroll.name}</h3>
      </div>
      
      {warscroll.profiles?.map((profile, idx) => (
        <div key={idx} className="warscroll-profile">
          <h4 className="profile-type">{profile.type}</h4>
          <div className="profile-stats">
            {Object.entries(profile).map(([key, value]) => {
              if (key === 'type' || key === 'name') return null;
              return (
                <div key={key} className="stat-row">
                  <span className="stat-key">{key}:</span>
                  <span className="stat-value">{value}</span>
                </div>
              );
            })}
          </div>
        </div>
      ))}

      {warscroll.abilities && warscroll.abilities.length > 0 && (
        <div className="warscroll-abilities">
          <h4><Zap size={16} /> Abilities</h4>
          <ul>
            {warscroll.abilities.map((ability, idx) => (
              <li key={idx}>{ability}</li>
            ))}
          </ul>
        </div>
      )}
    </div>
  );

  if (loading && !factionData) {
    return (
      <div className="loading-container">
        <Flame className="loading-icon spinning" size={48} />
        <p>Loading Age of Sigmar data...</p>
      </div>
    );
  }

  if (error) {
    return (
      <div className="error-container">
        <p className="error-message">{error}</p>
        <button onClick={fetchRepositoryData} className="retry-button">
          Retry
        </button>
      </div>
    );
  }

  return (
    <div className="aos-app">
      <header className="app-header">
        <div className="header-content">
          <Crown className="header-icon" size={36} />
          <h1>Age of Sigmar - Faction Pack</h1>
          <Crown className="header-icon" size={36} />
        </div>
      </header>

      {!factionData ? (
        <div className="faction-selector">
          <h2><Book size={24} /> Select a Faction</h2>
          <div className="faction-grid">
            {factions.map((faction, idx) => (
              <button
                key={idx}
                className="faction-button"
                onClick={() => fetchFactionData(faction)}
              >
                <Shield size={20} />
                <span>{faction.name}</span>
              </button>
            ))}
          </div>
        </div>
      ) : (
        <div className="faction-content">
          <div className="faction-header-banner">
            <button 
              onClick={() => setFactionData(null)} 
              className="back-button"
            >
              ← Back to Factions
            </button>
            <h2 className="faction-title">{factionData.name}</h2>
          </div>

          {loading ? (
            <div className="loading-container">
              <Flame className="loading-icon spinning" size={36} />
              <p>Loading faction details...</p>
            </div>
          ) : (
            <>
              {factionData.battleTraits.length > 0 && (
                <section className="content-section">
                  <h3 className="section-title">
                    <Swords size={24} /> Battle Traits
                  </h3>
                  <div className="cards-grid">
                    {factionData.battleTraits.map((trait, idx) => (
                      <AbilityCard key={idx} ability={trait} icon={Flame} />
                    ))}
                  </div>
                </section>
              )}

              {factionData.artefacts.length > 0 && (
                <section className="content-section">
                  <h3 className="section-title">
                    <Star size={24} /> Artefacts of Power
                  </h3>
                  <div className="cards-grid">
                    {factionData.artefacts.map((artefact, idx) => (
                      <AbilityCard key={idx} ability={artefact} icon={Star} />
                    ))}
                  </div>
                </section>
              )}

              {factionData.prayers.length > 0 && (
                <section className="content-section">
                  <h3 className="section-title">
                    <Book size={24} /> Prayer Lore
                  </h3>
                  <div className="cards-grid">
                    {factionData.prayers.map((prayer, idx) => (
                      <AbilityCard key={idx} ability={prayer} icon={Book} />
                    ))}
                  </div>
                </section>
              )}

              {factionData.spells.length > 0 && (
                <section className="content-section">
                  <h3 className="section-title">
                    <Zap size={24} /> Spell Lore
                  </h3>
                  <div className="cards-grid">
                    {factionData.spells.map((spell, idx) => (
                      <AbilityCard key={idx} ability={spell} icon={Zap} />
                    ))}
                  </div>
                </section>
              )}

              {factionData.warscrolls.length > 0 && (
                <section className="content-section">
                  <h3 className="section-title">
                    <Shield size={24} /> Warscrolls
                  </h3>
                  <div className="warscrolls-container">
                    {factionData.warscrolls.map((warscroll, idx) => (
                      <WarscrollCard key={idx} warscroll={warscroll} />
                    ))}
                  </div>
                </section>
              )}
            </>
          )}
        </div>
      )}

      <style jsx>{`
        .aos-app {
          font-family: 'Cinzel', 'Georgia', serif;
          background: linear-gradient(135deg, #1a1410 0%, #2d1810 100%);
          min-height: 100vh;
          color: #f5e6d3;
          padding-bottom: 2rem;
        }

        .app-header {
          background: linear-gradient(135deg, #8b4513 0%, #cd853f 50%, #8b4513 100%);
          border-bottom: 4px solid #d4af37;
          box-shadow: 0 4px 20px rgba(212, 175, 55, 0.3);
          padding: 2rem 1rem;
        }

        .header-content {
          max-width: 1200px;
          margin: 0 auto;
          display: flex;
          align-items: center;
          justify-content: center;
          gap: 1.5rem;
        }

        .header-content h1 {
          margin: 0;
          font-size: 2.5rem;
          color: #fff;
          text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
          letter-spacing: 2px;
          font-weight: 700;
        }

        .header-icon {
          color: #d4af37;
          filter: drop-shadow(0 0 10px rgba(212, 175, 55, 0.5));
        }

        .loading-container, .error-container {
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          min-height: 400px;
          gap: 1rem;
        }

        .loading-icon {
          color: #ff6b35;
        }

        .spinning {
          animation: spin 2s linear infinite;
        }

        @keyframes spin {
          from { transform: rotate(0deg); }
          to { transform: rotate(360deg); }
        }

        .error-message {
          color: #ff6b6b;
          font-size: 1.1rem;
        }

        .retry-button {
          padding: 0.75rem 1.5rem;
          background: #cd853f;
          border: 2px solid #d4af37;
          border-radius: 8px;
          color: white;
          font-size: 1rem;
          cursor: pointer;
          transition: all 0.3s;
        }

        .retry-button:hover {
          background: #d4af37;
          transform: translateY(-2px);
          box-shadow: 0 4px 12px rgba(212, 175, 55, 0.4);
        }

        .faction-selector {
          max-width: 1400px;
          margin: 2rem auto;
          padding: 0 1rem;
        }

        .faction-selector h2 {
          display: flex;
          align-items: center;
          gap: 0.5rem;
          color: #d4af37;
          font-size: 2rem;
          margin-bottom: 2rem;
          text-align: center;
          justify-content: center;
        }

        .faction-grid {
          display: grid;
          grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
          gap: 1.5rem;
        }

        .faction-button {
          background: linear-gradient(135deg, #3d2817 0%, #5a3d2b 100%);
          border: 2px solid #8b4513;
          border-radius: 12px;
          padding: 1.5rem;
          color: #f5e6d3;
          font-size: 1.1rem;
          cursor: pointer;
          transition: all 0.3s;
          display: flex;
          align-items: center;
          gap: 0.75rem;
          text-transform: capitalize;
          font-family: 'Cinzel', serif;
        }

        .faction-button:hover {
          background: linear-gradient(135deg, #5a3d2b 0%, #6d4c3b 100%);
          border-color: #d4af37;
          transform: translateY(-4px);
          box-shadow: 0 6px 20px rgba(212, 175, 55, 0.3);
        }

        .faction-content {
          max-width: 1400px;
          margin: 0 auto;
          padding: 0 1rem;
        }

        .faction-header-banner {
          background: linear-gradient(135deg, #8b4513 0%, #cd853f 100%);
          border-radius: 12px;
          padding: 2rem;
          margin: 2rem 0;
          border: 3px solid #d4af37;
          box-shadow: 0 4px 20px rgba(212, 175, 55, 0.2);
        }

        .back-button {
          background: rgba(0, 0, 0, 0.3);
          border: 2px solid #d4af37;
          color: white;
          padding: 0.5rem 1rem;
          border-radius: 6px;
          cursor: pointer;
          font-size: 0.9rem;
          margin-bottom: 1rem;
          transition: all 0.3s;
        }

        .back-button:hover {
          background: rgba(0, 0, 0, 0.5);
          transform: translateX(-4px);
        }

        .faction-title {
          margin: 0;
          font-size: 2.5rem;
          color: white;
          text-align: center;
          text-transform: uppercase;
          letter-spacing: 3px;
          text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }

        .content-section {
          margin: 3rem 0;
        }

        .section-title {
          display: flex;
          align-items: center;
          gap: 0.75rem;
          color: #d4af37;
          font-size: 2rem;
          margin-bottom: 1.5rem;
          padding-bottom: 0.5rem;
          border-bottom: 2px solid #8b4513;
        }

        .cards-grid {
          display: grid;
          grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
          gap: 1.5rem;
        }

        .ability-card {
          background: linear-gradient(135deg, #3d2817 0%, #5a3d2b 100%);
          border: 2px solid #8b4513;
          border-radius: 12px;
          padding: 1.5rem;
          transition: all 0.3s;
        }

        .ability-card:hover {
          border-color: #d4af37;
          transform: translateY(-4px);
          box-shadow: 0 6px 20px rgba(212, 175, 55, 0.2);
        }

        .ability-header {
          display: flex;
          align-items: center;
          gap: 0.75rem;
          margin-bottom: 1rem;
          padding-bottom: 0.75rem;
          border-bottom: 1px solid #8b4513;
        }

        .ability-header h4 {
          margin: 0;
          color: #ff6b35;
          font-size: 1.3rem;
        }

        .ability-icon {
          color: #d4af37;
        }

        .ability-content {
          color: #e8d4c0;
          line-height: 1.6;
        }

        .ability-section {
          margin: 0.75rem 0;
        }

        .ability-section strong {
          color: #d4af37;
          display: block;
          margin-bottom: 0.25rem;
        }

        .warscrolls-container {
          display: grid;
          grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
          gap: 2rem;
        }

        .warscroll-card {
          background: linear-gradient(135deg, #2d1810 0%, #3d2817 100%);
          border: 3px solid #8b4513;
          border-radius: 12px;
          padding: 1.5rem;
          transition: all 0.3s;
        }

        .warscroll-card:hover {
          border-color: #d4af37;
          transform: translateY(-4px);
          box-shadow: 0 8px 24px rgba(212, 175, 55, 0.3);
        }

        .warscroll-header {
          display: flex;
          align-items: center;
          gap: 1rem;
          margin-bottom: 1.5rem;
          padding-bottom: 1rem;
          border-bottom: 2px solid #d4af37;
        }

        .warscroll-header h3 {
          margin: 0;
          color: #ff6b35;
          font-size: 1.5rem;
        }

        .warscroll-icon {
          color: #d4af37;
        }

        .warscroll-profile {
          margin: 1.5rem 0;
          background: rgba(0, 0, 0, 0.2);
          padding: 1rem;
          border-radius: 8px;
          border: 1px solid #5a3d2b;
        }

        .profile-type {
          color: #cd853f;
          margin: 0 0 1rem 0;
          font-size: 1.1rem;
        }

        .profile-stats {
          display: grid;
          gap: 0.5rem;
        }

        .stat-row {
          display: flex;
          justify-content: space-between;
          padding: 0.5rem;
          background: rgba(139, 69, 19, 0.2);
          border-radius: 4px;
        }

        .stat-key {
          color: #d4af37;
          font-weight: 600;
        }

        .stat-value {
          color: #f5e6d3;
          font-family: 'Courier New', monospace;
        }

        .warscroll-abilities {
          margin-top: 1.5rem;
          padding-top: 1rem;
          border-top: 1px solid #8b4513;
        }

        .warscroll-abilities h4 {
          display: flex;
          align-items: center;
          gap: 0.5rem;
          color: #d4af37;
          margin: 0 0 0.75rem 0;
        }

        .warscroll-abilities ul {
          margin: 0;
          padding-left: 1.5rem;
          color: #e8d4c0;
        }

        .warscroll-abilities li {
          margin: 0.5rem 0;
          line-height: 1.5;
        }

        @media (max-width: 768px) {
          .header-content h1 {
            font-size: 1.5rem;
          }

          .faction-grid,
          .cards-grid,
          .warscrolls-container {
            grid-template-columns: 1fr;
          }

          .section-title {
            font-size: 1.5rem;
          }

          .faction-title {
            font-size: 1.8rem;
          }
        }
      `}</style>
    </div>
  );
};

export default AOSWarscrollsApp;
